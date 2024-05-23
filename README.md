# docker-pihole
Step-by-step instructions to get a Pi-hole up and running within a Docker container.
###### Note: This tutorial's instructions will be based on using a Raspberry Pi for the server, but you may still use these instructions to make your Pi-hole on an old PC, virtual machine, or hypervisor (Proxmox is a popular option).

# Materials
- Raspberry Pi (with power supply)
- MicroSD card
- Another PC to flash the OS and SSH to the Pi
- Ethernet (optional, but makes the build a few steps simpler)

# Step 1: Prepare the Raspberry Pi
### 1.1 Download the Raspberry Pi Imager on your PC.

For **Windows** and **Mac** users, navigate to: [https://www.raspberrypi.com/software/](https://www.raspberrypi.com/software/)

For **Lunix** users:

| Distro                                   | Command                           |
|------------------------------------------|-----------------------------------|
| **Ubuntu/Linux Mint/Debian and derivatives** | `sudo apt install rpi-imager` |
| **Arch/Manjaro** | `sudo pacman -S rpi-imager` |
| **CentOS/RHEL** | `sudo yum install rpi-imager` |
| **Fedora** | `sudo dnf install rpi-imager` |
| **OpenSUSE** | `sudo zypper install rpi-imager` |

### 1.2 Connect your MicroSD and start the Raspberry Pi Imager

  - CHOOSE DEVICE: [**Your Raspberry Pi**]
  - CHOOSE OS: **Raspberry Pi OS Lite (64-bit)**
  - CHOOSE STORAGE:[**Your MicroSD card**]
  - NEXT: "Would you like to apply OS customization settings?" **No, Continue**
