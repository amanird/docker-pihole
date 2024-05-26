# docker-pihole

Step-by-step instructions for building a Pi-hole DNS server inside a Docker container running Unbound on a Raspberry Pi.

*Note: These instructions are intended for using  a Raspberry Pi as the server. Alternatively, you could use an old PC, VM, or a hypervisor (such as Proxmox, which is a popular option).*

##### Auto-installer coming soon!

<br>

# Prerequisites
- PC
- microSD card (16GB or larger)
- Raspberry Pi (any) with power supply
- Ethernet
- additional keyboard and monitor (optional)

<br>

# Step 1: Prepare the Raspberry Pi
### Download the Raspberry Pi Imager on your PC

For **Windows** and **Mac** users, navigate to: [https://www.raspberrypi.com/software/](https://www.raspberrypi.com/software/)

For **Linux** users:

| Distro                                   | Command                           |
|------------------------------------------|-----------------------------------|
| **Ubuntu/Linux Mint/Debian and derivatives** | `sudo apt install rpi-imager` |
| **Arch/Manjaro** | `sudo pacman -S rpi-imager` |
| **CentOS/RHEL** | `sudo yum install rpi-imager` |
| **Fedora** | `sudo dnf install rpi-imager` |
| **OpenSUSE** | `sudo zypper install rpi-imager` |

<br>

### Connect your microSD to PC and start the Raspberry Pi Imager

*Note: I recommend ensuring your microSD is preformatted before using the Raspberry Pi Imager, as I've had errors in verification with cards that weren't formatted beforehand.*

- **CHOOSE DEVICE:**  **[Your Raspberry Pi]**

- **CHOOSE OS:**  **Raspberry Pi OS Lite (64-bit)** OR a Raspberry Pi OS Lite version compatible with your Pi

- **CHOOSE STORAGE:**  **[Your microSD card]**

- **NEXT:** "Would you like to apply OS customization settings?"  **No, Continue**

- **Use OS customisation? Would you like to apply OS customisation settings?:** [**EDIT SETTINGS**]

- **Set username and password** Check this box and proceed to make your login credentials.

- **Services** Check the Enable SSH box.

<br>

If you've successfully written and verified the OS flash,... **********************CONTINUE THIS LATER********* <br>

If you are using Wi-Fi instead of Ethernet we must pre-configure Wi-Fi (Ethernet users can ignore this). Go into the **bootfs** partition and create a **new empty file** named `wpa_supplicant.conf`

Paste this content into the file:
```
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
    
network={
ssid="your-network-ssid(name of the wifi)"
psk="your-network-password"
}
```

<br>
   
### Set up the Raspberry Pi

#### Insert the microSD card, connect to Ethernet, and turn on the Raspberry Pi
*You can **optionally** also connect your extra keyboard and monitor to the Raspberry Pi now and go through with the remaining setup from there, otherwise you can continue.*

<br>

Find the IP address of the Raspberry Pi from your router's admin interface. 
<br>

### SSH into the Raspberry Pi from a terminal on your PC: 
```
ssh pi@<your-pi-ip-address>
```

Default username: `pi`

Default password: `raspberry`

Change the default password using the `passwd` command.

<br>

# Step 2: Install Docker

### Update the system:
```
sudo apt update
sudo apt upgrade
```

```
sudo apt install -y docker.io
```

```
sudo systemctl enable docker
sudo systemctl start docker
```

```
sudo usermod -aG docker your-pi-username
```

<br>

# Step 3: Set Up Docker Compose
### Install Docker Compose:
```
sudo apt install -y docker-compose
```

```
docker --version
```

### Create Docker Compose File:
```
mkdir ~/pihole-unbound
cd ~/pihole-unbound
nano docker-compose.yml
```

#### Add the following to `docker-compose.yml`:
```
version: '3'

services:
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    environment:
      - TZ=America/New_York
      - WEBPASSWORD=your-pihole-password
      - DNS1=127.0.0.1#5335
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "67:67/udp"
      - "80:80"
      - "443:443"
    restart: always
    volumes:
      - pihole_config:/etc/pihole
      - dnsmasq_config:/etc/dnsmasq.d

  unbound:
    image: mvance/unbound:latest
    container_name: unbound
    restart: always
    volumes:
      - unbound_config:/opt/unbound/etc/unbound

volumes:
  pihole_config:
  dnsmasq_config:
  unbound_config:

```
Save and exit the nano file:

Exit: `Ctrl + X`

Save: `Y`

### Understanding the ports we're using:

| Ports          | Description                           |
|----------------|-----------------------------------|
| `53:53/tcp`    |DNS queries over TCP. This is used for larger DNS responses that cannot fit in a single UDP packet and for DNS zone transfers.|
| `53:53/udp`    |DNS queries over UDP. This is the primary method for DNS queries and responses due to its lower overhead compared to TCP.|
| `67:67/udp`    |DHCP server. This port is used for DHCP requests and responses, allowing the server to assign IP addresses to devices on the network.|
| `80:80`    |HTTP web interface. This port is used for accessing the Pi-hole admin interface over an unencrypted HTTP connection.|
| `443:443`    |HTTPS web interface. This port is used for accessing the Pi-hole admin interface over an encrypted HTTPS connection.|

<br>

# Step 4: Configure Pi-hole and Unbound
### Start the containers:
```
cd ~/pihole-unbound //necessary or no?
sudo docker-compose up -d
```

### Verify the contaiers are running:
```sudo docker ps```

<br>

# Step 5: Access Pi-hole admin interface to configure your network
Go to `http://<your-pi-ip-address>/admin` on your PC and log in using the password set in the `docker-compose.yml` file.

<br>

### Add to your blocklists

Navigate to the `Adlists` tab. You can find or create your own blocklists to suit your needs and preferences. I use several lists found on [https://firebog.net/](https://firebog.net/), a blocklist collection maintained by [WaLLy3K](https://github.com/WaLLy3K/wally3k.github.io) on GitHub.

After you've added all your blocklists, navigate to the `Tools` tab and then `Update Gravity` to update and save.

<br>

### Set Pi-hole as your primary DNS server for your network

Go to the `Settings` tab and then `DNS`. Uncheck any checked Upstream DNS Servers.

Add to **Custom 1 (IPv4)**: `127.0.0.1#5335`

`127.0.0.1#5335` *is a value that specifies the address and port of the primary DNS server. Here,* `127.0.0.1` *represents the localhost, and* `5335` *is the port number where the Unbound DNS server is running. This configuration tells Pi-hole to use the local Unbound DNS server as its primary DNS resolver.*

<br>

# Troubleshooting

Check Docker logs:
```
sudo docker logs pihole
sudo docker logs unbound
```

Restart containers:
```
sudo docker restart pihole
sudo docker restart unbound
```
