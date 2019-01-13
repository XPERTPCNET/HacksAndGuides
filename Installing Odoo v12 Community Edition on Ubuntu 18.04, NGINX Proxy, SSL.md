# Installing Odoo v12 Community Edition on Ubuntu 18.04, NGINX Proxy, SSL

**This tutorial assumes you have a working install of Ubuntu 18.04 and root access.**

## Prep for Odoo:
Let's login to your machine and get it ready fo Odoo.

*  Update Ubuntu:
```
sudo apt update && sudo apt-get dist-upgrade -y
```

* Install Git, Wget, Unzip, Pip, Python3 and dependancies:
```
sudo apt-get install wget git unzip python3-pip build-essential python3-dev python3-venv python3-wheel libxslt-dev libzip-dev libldap2-dev libsasl2-dev python3-setuptools 
```

* Install Odoo Web Dependencies:
```
sudo apt-get install -y npm
sudo npm install -g less less-plugin-clean-css
sudo apt-get install node-less
```
##  Install Wkhtmltox

In order to print PDF reports, you will need the wkhtmltopdf tool. The recommended version for Odoo is 0.12.1 which is not available in the official Ubuntu 18.04 repositories.

* Download package using wget and install:
```
wget https://builds.wkhtmltopdf.org/0.12.1.3/wkhtmltox_0.12.1.3-1~bionic_amd64.deb
sudo dpkg -i wkhtmltox_0.12.1.3-1~bionic_amd64.deb
```
If you get a dependency error:
```
sudo apt-get install -f
sudo dpkg -i wkhtmltox_0.12.1.3-1~bionic_amd64.deb
```
Proceed to complete install:
```
sudo ln -s /usr/local/bin/wkhtmltopdf /usr/bin
sudo ln -s /usr/local/bin/wkhtmltoimage /usr/bin
```

## Create user "odoo12" 
We will create a user with a Home Directory where we will later place the Odoo12 installation from GitHub:
```
sudo useradd -m -d /opt/odoo12 -U -r -s /bin/bash odoo12
```

## Install and Setup Odoo PostgreSQL Database

* Install PostgreSQL 10.5:
```
sudo apt-get install postgresql
```
* Set PostgreSQL master password:
```
sudo passwd postgres
```
### Create user odoo12 for PostgreSQL 
We are going to create a user similar to the one we previously created to run Odoo:
```
sudo su - postgres -c "createuser -s odoo12"
```

## Install and Configure Odoo 12

Now let's switch to odoo12 user:
```
sudo su - odoo12
```

Clone the GitHub Repository:
```
git clone https://www.github.com/odoo/odoo --depth 1 --branch 12.0 /opt/odoo12/odoo
```

### Install Python requirements for Odoo 12
We should be in the following directory:   (/opt/odoo12/)

- Issue the command cd /opt/odoo, and then create a new virtual environment with the command:

```
cd /opt/odoo12
python3 -m venv odoo12-venv
```
Active the environment with the command:
```
source odoo12-venv/bin/activate
pip3 install wheel
pip3 install -r odoo/requirements.txt
```
Optional: Here are additional requirements that may have been missed. There might be overlap but not a problem.
```
pip3 install Babel decorator docutils ebaysdk feedparser gevent greenlet html2text Jinja2 lxml Mako MarkupSafe mock num2words ofxparse passlib Pillow psutil psycopg2 pydot pyparsing PyPDF2 pyserial python-dateutil python-openid pytz pyusb PyYAML qrcode reportlab requests six suds-jurko vatnumber vobject Werkzeug XlsxWriter xlwt xlrd gdata
```

### Deactivate the environment, and exit back to the standard user with the commands:
```
deactivate
exit
```

### Setup Odoo Config
Create custom addons Directory:
```
su odoo12
mkdir /opt/odoo12/odoo-custom-addons
exit

```
Create the config file from sample included in GitHub:
```
sudo cp /opt/odoo12/odoo/debian/odoo.conf /etc/odoo12.conf
```
Edit the file:
```
sudo nano /etc/odoo12.conf
```
##
` /etc/odoo12.conf  `

 >  [options]
; Password that allows database operations:
; admin_passwd = my_admin_passwd
db_host = False
db_port = False
db_user = odoo12
db_password = False
addons_path = /opt/odoo12/odoo/addons,/opt/odoo12/odoo-custom-addons


# Create Odoo12 Service
To run Odoo as a service we need to create a service unit file in the /etc/systemd/system/ directory.
```
sudo nano /etc/systemd/system/odoo12.service
```
##
`/etc/systemd/system/odoo12.service `
> [Unit]
Description=Odoo12
Requires=postgresql.service
After=network.target postgresql.service
>
>[Service]
Type=simple
SyslogIdentifier=odoo12
PermissionsStartOnly=true
User=odoo12
Group=odoo12
ExecStart=/opt/odoo12/odoo12-venv/bin/python3 /opt/odoo12/odoo/odoo-bin -c /etc/odoo12.conf
StandardOutput=journal+console
>
>[Install]
WantedBy=multi-user.target

### Load the Odoo12 service
```
sudo systemctl daemon-reload
sudo systemctl start odoo12
```
Check status of Odoo 12 Service
```
sudo systemctl status odoo12
```
Should look something like this.....
![a2253feb.png](attachments\a2253feb.png)

Goto the website and fill out the new database creation form:

Use the IP of your server on port 8069:
> http://127.0.0.1:8069
