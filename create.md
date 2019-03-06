## @todo VirtualBox - Create a new virtual machine

* Install VirtualBox: https://virtualbox.org/

VirtalBox > New > (Expert Mode)

* Name: DockerVM
* Type: Linux
* Version: Ubuntu (64 bit)
* Memory size: 4096 MB
* Hard disk: Create harddisk
  - SATA Port 0, Hard disk, File location: System, 1000 GB, VDMK, Dynamically allocated

VirtualBox > Settings

* System
  - Motherboard > Boot Order: Optical, Hard Disk
  - Processor > CPU: 4 (Maximum of green)
  - Processor > Enable: PAE/NX
* Audio > Enable Audio: false
* Network
  - Adapter 1, NAT

### Download Debian

Download: Debian 9.8.0 - DVD 1

https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/

### Install Debian

* Graphical install or (Console) install (Console install is faster)
* Language: English (or what you prefer)
* Location: United States (or what you prefer)
* Keyboard: American English (or what you prefer)
* Hostname: dev-vm
* Domain name: (empty)

* Root Passwort: root
* Full username for new user: user
* Username for your account: user
* User passwort: user

* Clock/time zone: Central

* Partition disks: Guided - use entire disk
* Use: SCSI 3 (0,0,0) (sda) - 100.0 GB ATA VBOX HARDDISK
* Partitioning: All files in one partition
* Finish partitioning and write changes to disk
* Write changes to disk: Yes

* Scan another CD or DVD: No
* Use a network mirror: Yes
* Popularity-contest/Package use survey: No
* Software Selection: SSH Server, standard system utilities

* GRUB boot loader to master boot record: Yes
* Device boot loader installation: /dev/sda (...)
* Installation complete: Continue and reboot

## Procceed the installation

* Login as root

Configure APT and comment "deb cdrom: ..." to "#deb cdrom: ..." out.

```bash
nano /etc/apt/sources.list
```

Install Git, clone repository, go into the directory and run installation.

```Shell
# apt update
apt -y install git
git clone https://github.com/pluswerk/docker-vm.git
cd docker-vm
./create
```

Now you have created a new virtual machine.

I recommend to export the virtual machine to a ova file. See following...

## Virtualbox export

* Expert mode
* Virtualbox > File > Export Appliance
  - Open Virtualization Format 2.0

## Virtualbox import

To import a OVA file and configure the virtual machine see file [usage.md](usage.md).

### @todo Ubuntu 16.04 Server: Configure network interfaces

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
