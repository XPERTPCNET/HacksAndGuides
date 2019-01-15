# SaltStack Salt Quick Setup

## Install Salt Master with Minion
```
sudo wget -O bootstrap_salt.sh https://bootstrap.saltstack.com

# Add "-N" is you do not want to install minion:
sudo sh bootstrap_salt.sh -P -M
```
### Setup Master
#### Bind interface to salt service (optional)
> uncomment and change line "interface:" > save and exit
```
sudo systemctl stop salt-master
sudo nano /etc/salt/master
sudo systemctl start salt-master
```


### Setup Minion
#### Add salt-master to salt-minion config
 
- Set hostname
 > 
 ```
sudo hostnamectl set-hostname <saltmaster.example.com>

# Add dns name to hosts file:
sudo nano /etc/hosts

# Get hostname to place in minion config
sudo hostname
```
- Get Salt-Naster Public Key fingerprint to paste in minion config
```
sudo salt-key -F
# copy fingerprint after "master.pub:""
```
- Stop minion service and open config
```
sudo systemctl stop salt-minion
sudo nano /etc/salt/minion
```
> Once in the editor, find and uncomment line for salt-master, use your master hostname:
> "master: saltmaster.example.com"
>
>  Now find, uncomment, and paste master.pub key to:
>  "master_finger:  yourmaster.pubkey"

- Start salt-minion service
```
# Start Salt-Minion service
sudo systemctl start salt-minion
```
#### Add salt-minion to salt-master
- Check for unaccepted keys
```
sudo salt-key -L
```
> Your minion should show up under "Unaccepted Keys". Proceed to accept the minion


- Accept salt-minion key
```
sudo salt-key -a <minion-ID>

# Also accept all keys if you know all the minions:
sudo salt-key -A
```
- Check to see is you minion was accepted:
```
sudo salt-key -L
```
> The minion should no show up under "Accepted Keys:"
> 
---
## Salt-SSH Setup

### Prep the Saltless minion

#### Setup to run as sudo
We need to update our sudoers file so that Salt can run commands as sudo without issue.
```
sudo su -
visudo
```
> user    ALL=(ALL) NOPASSWD:ALL
If you can log in via SSH as root, you do not need to make these changes. Our cloud servers have this disabled. Turning off SSH root login is also generally considered a best practice.

#### Install Python
```
sudo apt-get install pyhton-minimal
```
### Prep the Salt-Master Server
Salt SSH is not installed alongside salt-master and is also not included in the bootstrap script. There are two wasy of installing salt:
 - Two methods exist:
### Prep the salt-master using debian package manager (easy)
```
sudo apt-get installl salt-ssh
```
### Prep the salt-master using pip 
```
pip install salt-ssh
```
#### Set Up the Salt SSH Roster
So how does Salt know what servers to SSH into? We need to use the /etc/salt/roster file, which was added after we installed Salt SSH. As with most of our configurations, the file is written in simple YAML.

To add a server to the roster, we first need to give the server an ID. Since our new minion was created specifically to work with Salt SSH, I just used a designation of ssh_minion; generally, however, this would be something more identifiable:
```
sudo nano /etc/salt/roster
```
> ssh-minion:
After this, we need to supply some parameters. At the very least, we need to always supply the host location (be sure to use your server's own private IP):
```
ssh-minion:
  host: <Host_IP>
  user: <user>
  sudo: True
  ```
  #### Setup ssh key to remotely connect to minion
  ```
  sudo ssh-copy-id -i /etc/salt/pki/master/ssh/salt-ssh.rsa.pub <user>@<PRIVATE IP OF SSH MINION>
  ```