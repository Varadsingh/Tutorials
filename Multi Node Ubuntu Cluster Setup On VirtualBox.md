## Multi Node Ubuntu Cluster Setup On VirtualBox

##### Step 0 - Create a virtual machine ( VM ) and install Ubuntu on it
> 1. Ctrl + N --> To create a new VM
> 2. RAM = 4 GB
> 3. Disk Space = 20 GB
> 4. Optical Drive = ubuntu-22.04-desktop-amd64.iso
> 5. VM Name = master

> Follow the installation instruction that come up on the screen till the installation is completed.
---
##### Step 1 - Install Dependencies
```bash
sudo usermod -aG sudo hadoop
sudo ufw disable
sudo apt update -y
sudo apt upgrade -y
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install make gcc linux-headers-$(uname -r) -y
```
---
##### Step 2 - VirtualBox Global Setting
On VirtualBox do the below steps
> File > Preferences > Network > Add New NAT Network ( Check "Supports DHCP" )
---
##### Step 3 - VirtualBox VM Setting
> Devices > Insert Guest Additions CD image
>
> Devices > Shared Clipboard > Bidirectional
>
> Devices > Drag and Drop > Bidirectional
>
> Devices > Shared Folders > Shared Folders Settings...
>
> Provide below details
> 1. Folder Path --> This is a location on host machine ( Windows )
> 2. Check "Auto-mount" and "Make Permanent"
> 3. Mount Point --> This is a location on virtual / guest machine ( Ubuntu )
---
##### Step 4 - Running the contents of the CD
```bash
sudo mkdir /media/cdrom
sudo mount /dev/cdrom /media/cdrom
sudo cd /media/cdrom
sh autorun.sh
```
---
##### Step 5 - Installing critical softwares
```bash
sudo usermod -aG vboxsf hadoop
sudo apt-get install vim -y
sudo apt-get install net-tools -y
sudo apt-get install ifupdown -y
sudo apt-get install openjdk-8-jdk -y
sudo apt install openssh-server openssh-client -y
sudo apt install python3.10-venv -y
```
---
##### Step 6 - Setting up Python Virtual Environment
```bash
python3 -m venv ~/.env
mkdir ~/pip
cd pip
nano pip.ini
```
```bash
# copy paste the below values
[global]
require-virtualenv = true
```
---
##### Step 7 - Create symbolic link for java
```bash
sudo ln -s /usr/lib/jvm/java-8-openjdk-amd64/ /usr/lib/
sudo mv /usr/lib/java-8-openjdk-amd64 /usr/lib/javahome
sudo poweroff
```
---
##### Step 8 - Add NAT Network To VM
> 1. Go to VM settings > Network > Adapter
> 2. Check "Enable Network Adapter"
> 3. Attach to "NAT Network"
> 4. Name --> Chose the one created in Step 2
---
##### Step 9 - Update Network Manager configuration file in master node
```bash
# Edit the configuration file
sudo nano /etc/netplan/01-network-manager-all.yaml
```
```bash
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp0s3:
      dhcp4: yes
    enp0s8:
      dhcp4: no
      addresses: [192.168.1.101/24]
	  # 192.168.1.101 --> for master node
	  # 192.168.1.102 --> for slave1 node
	  # 192.168.1.103 --> for slave2 node
```
```bash
# validate the configuration file
sudo netplan try

# apply the configuration file
sudo netplan apply
```
---
##### Step 10 - Update /etc/hosts
```bash
sudo nano /etc/hosts
```
```bash
# copy paste the below line to the end of the file

192.168.1.101   master
192.168.1.102   slave1
192.168.1.103   slave2
```
---
##### Step 11 - Shutdown the VM
```bash
sudo poweroff
```
---
##### Step 12 - Clone the VM
> In VirtualBox, right-click on the VM > Clone...
> Provide a name and use all the default setting.
> Create a Full Clone.
> Create 2 clones one for slave1 and another for slave2
> Might take 10 - 15 mins per clone
---
##### Step 13 - Update Network Manager configuration file in slave nodes
> Follow Step 9, but replace the addresses as mentioned for the slave nodes
> Restart all the nodes after completion
---
##### Step 14- Ping the VM's in the cluster
> 1. Start all the VM's and run the below commands to confirm that the VM's are communicating with each other
```bash
ping master
ping slave1
ping slave2
```
