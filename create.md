## VirtualBox - Create a new virtual machine

* Install VirtualBox: https://virtualbox.org/

VirtalBox > New > (Expert Mode)

* Name: DockerVM
* Type: Linux
* Version: Ubuntu (64 bit)
* Memory size: 4096 MB
* Hard disk: No, do not add a virtual hard disk

VirtualBox > Settings

* System
  - Motherboard > Boot Order: Optical, Hard Disk
  - Processor > CPU: 4 (Maximum of green)
* Storage (Expert mode)
  - SATA Port 0, Hard disk, File location: System, 100 GB, VDMK, Dynamically allocated
  - SATA Port 2, Compact Disc, Insert operating system
* Audio > Enable Audio: false
* Network (NAT or Bridged Adapter)
  - Adapter 1, NAT
  - Adapter 2, Bridged Adapter, eth1 (Your network device)

### Download Ubuntu

Ubuntu Server 18.04 or 16.04 (64 bit).

https://ubuntu.com/

### Installation for Ubuntu Server 16.04

* Language: English (or what you prefer)
* Install Ubuntu Server

* Language: English
* Country: United States
* Optional: Detect Keyboard Layout
* Select Primary Network interface

* Hostname: dev-vm
* Full name: user
* Username: user
* Passwort: user

* Encrypt home directory: No (Your preference)
* Time zone: Europe/Berlin (Your preference)
* Partition method: Guided - use the entire disk
  - SCSI1 (0,0,0) (sda) - 80 GB ATA VBOX HARDDISK

* HTTP proxy: (empty)
* No automatic updates
* Software
  - standard system utilities
  - OpenSSH server
* Install GRUB boot loader = Yes
* Reboot

### Installation for Ubuntu Server 18.04

* Page 1 - Language:
  - Language: English (or what you prefer)
* Page 2 - Keyboard configuration:
  - Your preference -> Identify keyboard
  - Layout: English (US)
  - Variant: English (US)
* Page 3 - Ubuntu 18.04:
  - Install Ubuntu
* Page 4 - Network connections:
  - Network configuration
* Page 5 - Configure proxy:
  - Proxy address: (empty)
* Page 6 - Configure Ubuntu archive mirror:
  - (bypass)
* Page 7 - Filesystem setup:
  - Use an Entire Disk
  - VBOX_HARDDISK_...
* Page 8 - Profile:
  - Your name: user
  - Your Server's name: dev-vm
  - Username: user
  - Passwort: user
* Page 9 - Feature Server: (bypassed)
* Page 10: (Installation running)
* Page 11: (Reboot)

## Procceed the installation

Set password for root account with password: root

```Shell
sudo passwd root
```

### Ubuntu 16.04 Server: Configure network interfaces

```Shell
ip a
sudo vim /etc/network/interfaces
```

File: interfaces

```Shell
# The loopback network interface
auto lo
iface lo inet loopback

# Network 1 (NAT)
auto enp0s3
iface enp0s3 inet dhcp

# Network 2 (Bridged Adapter)
auto enp0s8
iface enp0s8 inet dhcp

# Network 3 (Host-only Adapter)
#auto enp0s9
#iface enp0s9 inet dhcp
```

Restart network interfaces.

```Shell
sudo ifdown -a && sudo ifup -a
```

### Repository

Install Git, clone repository and go into the directory.

```Shell
sudo apt -y install git
git clone https://github.com/Cyb10101/dev-vm-linux.git /home/user/docker-vm
```

### Install

Run installation.

```Shell
sudo ./create
```

Note:

* Ubuntu 18.04: Configuring console-setup: UTF-8

Now you have created a new virtual machine.

I recommend to export the virtual machine to a ova file. See following...

## Virtualbox export

* Expert mode
* Virtualbox > File > Export Appliance
  - Open Virtualization Format 2.0

## Virtualbox import

To import a OVA file and configure the virtual machine see file [usage.md](usage.md).
