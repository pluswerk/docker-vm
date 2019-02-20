# Virtualbox import - General

Here is explained step by step how to import the VirtualBox OVA file and configure the virtual machine.

## Import OVA file

First import the VirtualBox OVA file.
I recommend to switch on the expert mode, because my explanation builds on it.

* Virtualbox > File > Import Appliance
* Click on button "Expert mode"
* Choose your OVA file and import it
* Reinitialize the MAC address of all network cards = true (Create new MAC Addresses)

If the OVA has been imported, the virtual machine has to be adjusted a bit.

Create a network for host-only adapter, if not exists one:

* VirtualBox > File > Host Network Manager
* Network > Create

Change the settings in the virtual machine and add a new second hard disk.
The first virtual hard disk with the system should be on the system disk (SSD).
The second virtual disk with the data should be on a second hard disk (HDD).
If snapshots are created, they may also be stored on the second hard disk (HDD).

VirtualBox > Machine > Settings

* General
  - Advanced > Snapshot Folder: (Only if snapshots are created. Best on second hard drive.)
* System
  - Motherboard > Memory: 2048 or 4096 MB (Server, Desktop or what you like)
  - Processor > CPU: 4 (Maximum of green)
* Storage
  - Create new hard disk > Expert mode
    - File location: Data (Should be on your biggest drive)
    - File size: 1TB (Maximum of available space)
    - Hard disk file type: VMDK
    - Storage on physical hard disk: Dynamically allocated
    - Sata Port 1
* Network
  - Configure Adapter 1 (Just get Internet connection)
    - Enable Network Adapter = True
    - Attached to: NAT
    - Adapter Type: Intel PRO/1000 MT Desktop (82540EM)
    - Generate new MAC Address
  - Configure Adapter 2 (A new computer in your Network)
    - Enable Network Adapter = True
    - Attached to: Bridged Adapter
    - Name: (Your host network card)
    - Adapter Type: Intel PRO/1000 MT Desktop (82540EM)
    - Generate new MAC Address
  - Configure Adapter 3 (I don't need a Network, only host to guest)
    - Enable Network Adapter = True
    - Attached to: Host-only Adapter
    - Name: (Virtualbox)
    - Adapter Type: Intel PRO/1000 MT Desktop (82540EM)
    - Generate new MAC Address

### Explanation of the network configuration

This is a simple configuration of the network and theoretically should work well on all machines.
Depending on usage and system, this should be personalized.

Adapter 1, Just open a connection to the internet. You can not access directly though.

Adapter 2, To come from another computer into the virtual machine.

Adapter 3, To get from the host directly to the virtual machine.
This is useful when wired and WIFI connection is switched.

## Configure virtual machine

Start the virtual machine.

The instructions are designed to do everything with the user "user".
In general, you should never need the user "root".

### Passwords

The passwords of Linux users are the same as the user names.
Note: You can't connect your ssh with the user "root".

* Username: root | Password: root
* Username: user | Password: user


## Run configuration script

```Shell
sudo /home/user/docker-vm/usage
```

### Minimal configuration

* Step 1: Configure system

Follow the instructions in script.

* Step 2: Create second harddisk

The second hard disk is for your Projects and Docker data.
This allows you to variably adjust the storage space and even move it to another hard drive in the host system.

Partition and format the second hard disk in terminal or with gparted.

Format the second hard disk by using the usage script "Create second harddisk".

### Optional configuration

* Step 3: Message of the day

* Step 4: Change keyboard layout

* Step 5: Convert SSH key to Putty key

* Step 6/7: Change login shell root/user

Change permanently from "bash" to "zsh".

## Configure Network

I recommend configuring the network because it can be different for everyone.

With the command "ip a" you can check your current network configuration.

Check if the interfaces "en0s8" are assigned to the correct MAC addresses.
If that's not the case, you'll need to adjust those values.

```Shell
ip a
```

### Optional Ubuntu 18.04 Server: Set static IP in network configuration

Edit Netplan file.

```Shell
ls /etc/netplan
sudo vim /etc/netplan/50-cloud-init.yaml
```

Change enp0s8 in 50-cloud-init.yaml.

```yaml
network:
    version: 2
    ethernets:
        enp0s3:
            addresses: []
            dhcp4: true
            optional: true
            nameservers:
                addresses: [127.0.1.1, 8.8.8.8]
        enp0s8:
            addresses: [192.168.178.50/24]
            gateway4: 192.168.178.1
            nameservers:
                addresses: [127.0.1.1, 8.8.8.8]
        enp0s9:
            addresses: [192.168.56.101/24]
            gateway4: 192.168.56.1
            nameservers:
                addresses: [127.0.1.1, 8.8.8.8]

        # Maybe better for Host-only Adapter
        enp0s9:
            addresses: [192.168.56.101/24]
            dhcp4: no
```

Restart network.

```Shell
sudo netplan generate && sudo netplan apply
```

### Optional Other Ubuntu: Set static IP in network configuration

Get and configure interfaces.

```Shell
sudo vim /etc/network/interfaces
```

Adapt file interfaces. Example configuration:

```text
# The loopback network interface
auto lo
iface lo inet loopback

# Network 1 (NAT)
auto enp0s3
iface enp0s3 inet dhcp

# Network 2 (Bridged Adapter)
auto enp0s8
iface enp0s8 inet static
  address 192.168.178.50
  netmask 255.255.255.0
  gateway 192.168.178.1

# Network 3 (Host-only Adapter)
auto enp0s9
iface enp0s9 inet static
  address 192.168.56.101
  netmask 255.255.255.0

  # Maybe not needed because it works automatically
  #network 192.168.56.0
  #broadcast 192.168.56.255

  # Is not needed because there is no internet
  #gateway 192.168.56.1
```

Reconnect network card, restart in case of doubt.

```Shell
sudo ifdown -a && sudo ifup -a
```

### Message of the day - Keep me from working

Yes, you wan't it! Your boss would kill you, but your soul thanks you.
Each new Terminal you opening on desktop or with ssh, you get a new message.

To activate/deactivate it use usage script "Message of the day".

Show which fortunes are available and configure it at function "terminalMotd" in .shell-methods file:

```Shell
ls /usr/share/games/fortunes
vim /home/user/.shell-methods
```

## Documentation

What can i do?

### Connect file browser to virtual machine

The /home/user/projects can be accessed via Windows sharing or SSH.
This will allow you to easily develop an IDE.

#### SSHFS share - Linux configuration

Note: If you are running Linux, you don't wan't this virtual machine. ;-)

Connect over SSH from Linux host to Linux guest.
IP address must be adapted for the virtual machine.

Run in a terminal window:

```Shell
sudo apt -y install sshfs
sudo mkdir -p /mnt/ssh/projects

vim ~/.bashrc && vim ~/.zshrc
```

Append to file ~/.bashrc and ~/.zshrc:

```Shell
alias vm-on='sshfs -o IdentitiesOnly=yes -o compression=no -o cache=yes -o kernel_cache -o allow_other -o IdentityFile=~/.ssh/id_rsa -o idmap=user -o uid=1000 -o gid=1000 user@192.168.56.101:/mnt/data/home/user/projects /mnt/ssh/projects'

alias vm-off='fusermount -u /mnt/ssh/projects'
```

Add 'user_allow_other' in file /etc/fuse.conf:

```Shell
sudo vim /etc/fuse.conf
```

#### Samba/Windows share

Connect over Samba from Windows/Linux host to Linux guest.
The folder /var/www is accessible over Samba share "www".
IP address (192.168.56.101) must be adapted for the virtual machine.

##### Samba/Windows share - Windows configuration

Run in a command window:

```Shell
net use Z: \\192.168.56.101\projects /PERSISTENT:YES
```

###### Bugfix Windows insecure guest access

If you do not get access via Samba you have to configure the following Group Policy settings:

```text
Windows Group Policy:
Computer configuration\administrative templates\network\Lanman Workstation
"Enable insecure guest logons" = true
```

[Guest access in SMB2 disabled by default](https://support.microsoft.com/en-us/help/4046019/guest-access-in-smb2-disabled-by-default-in-windows-10-and-windows-ser)

##### Samba/Windows share - Linux configuration

Run in a terminal window:

```Shell
sudo apt -y install cifs-utils
sudo mkdir -p /mnt/samba/projects
sudo chmod 777 /mnt/samba/projects

sudo /etc/fstab
```

Append to file /etc/fstab with your virtual machine IP:

```Text
//192.168.56.101/projects /mnt/samba/vm cifs uid=localUsername,username=user,password=user 0 0
```

### DNS Server (example.vm)

You can use DNS Server for the example.vm domains to work.
IP address (192.168.56.101) must be adapted for the virtual machine.

#### DNS Server - Linux Ubuntu Desktop

```Shell
sudo apt -y install resolvconf
sudo vim /etc/NetworkManager/NetworkManager.conf
```

Append to file: /etc/NetworkManager/NetworkManager.conf

```ini
[main]
dns=dnsmasq
```

Run these commands in the shell:

```Shell
sudo sh -c 'echo "nameserver 127.0.1.1" >> /etc/resolvconf/resolv.conf.d/head'
sudo sh -c 'echo "nameserver 8.8.8.8" >> /etc/resolvconf/resolv.conf.d/head'
sudo sh -c 'echo "address=/.vm/192.168.56.101" >> /etc/NetworkManager/dnsmasq.d/development'
sudo systemctl restart network-manager
sudo resolvconf -u
```

#### Windows DNS Server (Acrylic DNS Proxy)

Download and install: http://mayakron.altervista.org/wikibase/show.php?id=AcrylicHome

Start menu > Control Panel > Network and Internet > Network and Sharing Center > Change adapter settings

Edit network > Internet Protocol Version 4 (TCP/IPv4) > Properties > Use the following DNS server addresses:

* Preferred DNS server: 127.0.0.1

Start menu > Acrylic DNS Proxy > Acrylic UI

* File > Open Acrylic Hosts

Append with your virtual machine IP:

```Text
192.168.56.101 /.*\.vm$
```

Maybe you must restart the Service:

* Actions > Restart Acrylic Service

#### Configure xDebug in PhpStorm

Go to the menu:

PhpStorm > Run > Start Listening for PHP Debug Connections

then reload website.
PhpStorm should responds automatically and ask if your want to add a server.

##### Configure xDebug in PhpStorm - Add Server and path mapping

For some projects you must add a server and a path mapping.

PhpStorm > Run > Edit Configurations > (+) Add > PHP Remote Debug
- Filter debug connection by IDE key: true
- Add a Server (...) - Example: Nginx https
  - Name: www.example.vm:443
  - Host: www.example.vm
  - Port: 443
  - Use path mappings = true
  - Project files/mnt/projects/example/www = /mnt/data/home/user/projects/example/www (Connection between host and guest system)
- Add a Server (...) - Example: Apache https
  - Name: www.example.vm:4430
  - Host: www.example.vm
  - Port: 4430
  - Use path mappings = true
  - Project files/mnt/projects/example/www = /mnt/data/home/user/projects/example/www (Connection between host and guest system)
- IDE key (Session ID): PHPSTORM

@todo
Make this unnecessary: xdebug.remote_host=192.168.56.1
Would not work anyway because of other settings.

@deprecated Maybe not. Works with symfony
PhpStorm > Run > Edit Configurations > Defaults > PHP Remote Debug
- Name: example_www (Just a name)
