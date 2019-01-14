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

