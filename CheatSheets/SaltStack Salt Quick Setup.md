# SaltStack Salt Quick Setup

## Install Salt Master with Minion
```
sudo wget -O bootstrap_salt.sh https://bootstrap.saltstack.com

# Add "-N" is you do not want to install minion:
sudo sh bootstrap_salt.sh -M
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