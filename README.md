# docker-pihole

Step-by-step instructions for building a Pi-hole DNS server inside Docker running Unbound all on a Raspberry Pi.

*Note: These instructions will be based on using a Raspberry Pi microcomputer as the server. Alternatively you may use an old PC, virtual machine, or hypervisor (Proxmox is a popular option).*

##### Auto-installer coming soon!

<br>

# Materials
- Raspberry Pi (with power supply)
- MicroSD card (16GB or larger)
- Another PC to flash the OS and SSH to the Pi
- Ethernet
- Keyboard and Monitor (optional)

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

### Connect your MicroSD to PC and start the Raspberry Pi Imager
CHOOSE DEVICE:  **[Your Raspberry Pi]**

CHOOSE OS:  **Raspberry Pi OS Lite (64-bit)** **OR** a Raspberry Pi OS Lite version compatible with your machine of choice

CHOOSE STORAGE:  **[Your MicroSD card]**

NEXT: "Would you like to apply OS customization settings?"  **No, Continue**

<br>

After successfully writing the OS to the MicroSD, there will now be a **bootfs** partition and a **rootfs** partition. Go into the **bootfs** partition and create a **new empty file** named `ssh`.

<br>

If you are using Wi-Fi instead of Ethernet we must configure Wi-Fi (Ethernet users can ignore this). Go into the **bootfs** partition and create a **new empty file** named `wpa_supplicant.conf`

Paste this content into the file:
```
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
    
network={
ssid="your-network-ssid"
psk="your-network-password"
}
```

<br>
   
### Set up the Raspberry Pi

#### Insert the MicroSD card, connect to Ethernet, and turn on the Raspberry Pi
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

### Install Docker:
`
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker pi
`

### Reboot
`sudo reboot`

<br>

# Step 3: Set Up Docker Compose
### Install Docker Compose:
`
sudo apt install python3-pip
sudo pip3 install docker-compose
`

### Create Docker Compose File:
`
mkdir ~/pihole-unbound
cd ~/pihole-unbound
nano docker-compose.yml
`

#### Add the following to `docker-compose.yml`:
```
version: '3'

services:
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    environment:
      - TZ=America/New_York
      - WEBPASSWORD= your-strongest-pihole-password
      - DNS1=127.0.0.1#5335
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "67:67/udp"
      - "80:80"
      - "443:443"
    restart: always

  unbound:
    image: mvance/unbound:latest
    container_name: unbound
    restart: always
```
Save and exit the nano file:

Exit: `Ctrl + X`

Save: `Y`

### Understanding the ports we're using:

| Ports          | Description                           |
|----------------|-----------------------------------|
| `53:53/tcp`    |DNS queries over TCP. This is used for larger DNS responses that cannot fit in a single UDP packet and for DNS zone transfers.|
| `53:53/udp`    |DNS queries over UDP. This is the primary method for DNS queries and responses due to its lower overhead compared to TCP.|
| `67:67/udp`    |DHCP server. This port is used for DHCP (Dynamic Host Configuration Protocol) requests and responses, allowing the server to assign IP addresses to devices on the network.|
| `80:80`    |HTTP web interface. This port is used for accessing the Pi-hole admin interface over an unencrypted HTTP connection.|
| `443:443`    |HTTPS web interface. This port is used for accessing the Pi-hole admin interface over an encrypted HTTPS connection.|

<br>

# Step 4: Configure Pi-hole and Unbound
### Start the containers:
`
cd ~/pihole-unbound
sudo docker-compose up -d
`

### Verify the contaiers are running:
`sudo docker ps`

<br>

# Step 5: Access Pi-hole admin interface to configure your network
Go to `http://<your-pi-ip-address>/admin` on your PC and log in using the password set in the `docker-compose.yml` file.

<br>

### Add to your blocklists

Navigate to the `Adlists` tab. You can find or create your own blocklists to suit your needs and preferences. I use several lists from @WaLLy3K on GitHub at their website [https://firebog.net/](https://firebog.net/).

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
