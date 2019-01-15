# KVM server with Ubuntu 18.04

Kernel-based Virtual Machine (KVM) is a linux based virtualization technology tha turns your linux server into a hypervisor. It is light, stable, flexible and free!

## Check if your CPU supports KVM

### Install and run CPU Checker:
```
sudo apt-get instll cpu-checker
kvm-ok
```
expected output:
>  KVM acceleration can be used 

- If your CPU is ready then proceed to install KVM. If it doesn not pass, then you can proceed but don't expect good performance.
- If you're trying nested VM using a hypervisor then you have to enable it for the VM you are installing KVM on. Most Hypervisor clients allow you to edit the VM with the GUI, for Hyper-V manager this si not possible, try this Poweshell command (requires VM shutdown):
```
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
Replace <VMName> with your VM instance name.

##  Install KVM server
In this install we're going minimal. But some users will also want virt-manager for a GUI. So just add it to the end of the install command.
```
sudo apt instal qemu qemu-kvm qemu-utils libvirt-clients libvirt-daemon-system python-vm-builder bridge-utils virtinst
```
Installed components:
- qemu: fast CPU emulator.
- qemu-kvm: Kernel-based Virtual machine, QEMU uses it for CPU virtualization.
- qemu-utils: provides QEMU related utilities (qemu-img, qemu-io, qemu-nbd)
- libvirt-deamon-system:  C toolkit to interact with the virtualization capabilities.
- libvirt-clients:  This package contains the libvirt shell virsh and other client binaries.
- python-vm-builder: powerful command line tool for building virtual machines.
- bridge-utils provides a bridge from your network to the virtual machines.
- virtinst: Programs to create and clone virtual machines.
- virt-manager (optional): GUI manager for you Linux desktop. (we are using Kimchi below)

### Try Virsh commands on KVM:
```
sudo virsh nodeinfo
```
Results example: (depend on your Hw or VM)
> CPU model:           x86_64
CPU(s):             4
CPU frequency:       2393 MHz
CPU socket(s):       1
Core(s) per socket:  1
Thread(s) per core:  2
NUMA cell(s):        1
Memory size:         11982500 KiB

## Install  Kimchi and Wok

### Install required components:
```
sudo apt install libvirt-bin python-paramiko python-pil novnc python-libvirt python-ethtool python-ipaddr python-guestfs libguestfs-tools spice-html5 spice-html5 python-magic keyutils libnfsidmap2 libtirpc1 nfs-common rpcbind python-configobj python-parted -y
sudo apt install nginx -y
```
### Install Wok and Kimchi
```
wget https://github.com/kimchi-project/kimchi/releases/download/2.5.0/wok-2.5.0-0.noarch.deb
wget https://github.com/kimchi-project/kimchi/releases/download/2.5.0/kimchi-2.5.0-0.noarch.deb
```
### Install dependancies
```
sudo dpkg -i wok-2.5.0-0.noarch.deb

sudo apt install -f -y

sudo dpkg --ignore-depends=python-imaging -i kimchi-2.5.0-0.noarch.deb

#reboot here
sudo shutdown -r now

#If you have UFW enabled please add the port
sudo ufw allow 8001/tcp
```
Now open the web interface on

> https://Server_IP:8001

Type your linux username and password.

**WAIT!!!!**
There are errors with the dependencies for wok and issues with iso iimages that will cause template errors... you will needs these fixes:


# FIXES for Kimchi 
## Problem with Kimchi Package Dependancies

Python-imaging was replaced with Python-pil and is no longer needed. Please removed to avoid the apt command from uninstalling Kimchi due to unmet dependancies.

```
sudo nano /var/lib/dpkg/
```
>Search for "Package: kimchi".
  On line with "Depends:" clause, replace python-imaging with python-pil.
  Save file.

## Fix Bug with Template creation with Ubuntu 18.04 Template

 Replace bugy file with patched file from github:
```
sudo wget -q https://raw.githubusercontent.com/kimchi-project/kimchi/1ec059af4040c50b1a7b9a34253510a46ca09d3b/model/templates.py -O /usr/lib/python2.7/dist-packages/wok/plugins/kimchi/model/templates.py 
# Reboot after file change
sudo shutdown -r now
```


OK ENJOY!