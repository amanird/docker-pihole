# docker-pihole
Step-by-step instructions to get a Pi-hole up and running within a Docker container.
###### Note: This tutorial's instructions will be based on using a Raspberry Pi microcomputer as the server. Alternatively you may use an old PC, virtual machine, or hypervisor (Proxmox is a popular option).

## Table of Contents
- [Introduction](#introduction)
- [Materials](#materials)
- [Installation](#installation)
  - [Step 1: Prepare the Raspberry Pi](#step-1-prepare-the-raspberry-pi)
  - [Step 2: Connect via SSH](#step-2-connect-via-ssh)
  - [Step 3: Update and Upgrade](#step-3-update-and-upgrade)
  - [Step 4: Install Docker](#step-4-install-docker)
  - [Step 5: Create a Docker Network](#step-5-create-a-docker-network)
  - [Step 6: Run Pi-hole and Unbound Containers](#step-6-run-pi-hole-and-unbound-containers)
  - [Step 7: Access Pi-hole Web Interface](#step-7-access-pi-hole-web-interface)
- [Advanced Configuration](#advanced-configuration)
- [Troubleshooting](#troubleshooting)

# Materials
- Raspberry Pi (with power supply)
- MicroSD card
- Another PC to flash the OS and SSH to the Pi
- Ethernet (optional, but makes the build a few steps simpler)
- Keyboard and Monitor (optional)

# Step 1: Prepare the Raspberry Pi
### 1.1 Download the Raspberry Pi Imager on your PC.

For **Windows** and **Mac** users, navigate to: [https://www.raspberrypi.com/software/](https://www.raspberrypi.com/software/)

For **Linux** users:

| Distro                                   | Command                           |
|------------------------------------------|-----------------------------------|
| **Ubuntu/Linux Mint/Debian and derivatives** | `sudo apt install rpi-imager` |
| **Arch/Manjaro** | `sudo pacman -S rpi-imager` |
| **CentOS/RHEL** | `sudo yum install rpi-imager` |
| **Fedora** | `sudo dnf install rpi-imager` |
| **OpenSUSE** | `sudo zypper install rpi-imager` |

### 1.2 Connect your MicroSD to PC and start the Raspberry Pi Imager
  - CHOOSE DEVICE:  **[Your Raspberry Pi]**
  - CHOOSE OS:  **Raspberry Pi OS Lite (64-bit)** **OR** a Raspberry Pi OS Lite version compatible with your machine of choice
  - CHOOSE STORAGE:  **[Your MicroSD card]**
  - NEXT: "Would you like to apply OS customization settings?"  **No, Continue**
  <br>
  
After successfully writing the OS to the MicroSD, there will now be a **bootfs** partition and a **rootfs** partition...
  - Go into the **bootfs** partition and create a **new empty file** named `ssh`
<br>

If you are using Wi-Fi instead of Ethernet we must configure Wi-Fi (Ethernet users can ignore this)
  - Go into the **bootfs** partition and create a **new empty file** named `wpa_supplicant.conf`
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
   
### 1.3 Set up the Raspberry Pi

  - Insert the MicroSD card into the Raspberry Pi
  - Connect the Ethernet cable, power supply, and power on the Raspberry Pi
  - You can **optionally** connect your extra keyboard and monitor to the Raspberry Pi and go through with the remaining setup from there, otherwise continue
  - Note down the IP address of the newly connected Raspberry Pi
    - You can find this in your home router's admin page.
<br>

  





















# FAQ
### Why use Docker instead of just installing Pi-hole in the regular OS?
I found that the number of steps and complexity is less when using Docker's built-in Pi-hole architecture.

### Why are we using the Lite version of Raspberry Pi OS?
For server applications like Pi-hole, a GUI is unnecessary. Without the overhead of a GUI, a headless OS uses less resources, has a smaller attack surface, and avoids needless bloatware.
