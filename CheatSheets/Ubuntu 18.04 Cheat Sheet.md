# Ubuntu 18.04 Cheat Sheet

## Update Ubuntu
```
sudo apt update &&  sudo apt dist-upgrade -y
```
## Change Hostname
```
sudo nano /etc/cloud/cloud.cfg
## set: preserve_hostname: true
## save and exit

sudo nano /etc/hosts
## add <Name> to line for 127.0.0.1
## save and exit

sudo hostnamectl set-hostname <Name>
```
## Stop asking sudo password for a user
```
sudo visudo
```

> Visudo is a tool used to edit the configuration file of sudo . The command will prompt your for your password. Enter it one last time. The configuration file will open in a text editor, most likely Nano. Scroll down to the end of the document and add the following line:

 username ALL=NOPASSWD: ALL


