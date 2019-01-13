# Migrate Odoo 10 to 11

[Upgrade Odoo 10 to 11 Using OpenUpgrade – BIROINFOTEK](http://biroinfotek.com/odoo-upgrade-10-to-11-using-openupgrade/)

---

 HomeKnowledge BaseOdooUpgrade Odoo 10 to 11 Using OpenUpgrade
UPGRADE ODOO 10 TO 11 USING OPENUPGRADEOdoo  April 16, 2018  0  taf 
Steps:

Create directory openupgrade: $ mkdir /var/tmp/openupgrade
Open the folder : $ cd /var/tmp/openupgrade
Download migrate.py from module openupgrade odoo 11: $ wget https://raw.githubusercontent.com/OCA/OpenUpgrade/11.0/scripts/migrate.py
Moved odoo-server.conf to the same folder as migrate.py: $ sudo cp /etc/odoo-server.conf /var/tmp/openupgrade
Run migrate.py: python migrate.py –config=odoo-server.conf –database=mydb –run-migrations=10.0,11.0
Restore mydb_migrated to odoo 11 environment
See migration log for errors: $ nano /var/tmp/openupgrade/migration.log
Output step 5:

—Start—

python migrate.py –config=odoo-server.conf –database=bes –run-migrations=10.0,11.0
linking server/addons to /var/tmp/openupgrade/10.0/addons
getting git://github.com/OpenUpgrade/OpenUpgrade.git
Cloning into ‘/var/tmp/openupgrade/10.0/server’…
remote: Counting objects: 24236, done.
remote: Compressing objects: 100% (19609/19609), done.
remote: Total 24236 (delta 7736), reused 9786 (delta 3704), pack-reused 0
Receiving objects: 100% (24236/24236), 110.96 MiB | 689.00 KiB/s, done.
Resolving deltas: 100% (7736/7736), done.
Checking connectivity… done.
Checking out files: 100% (21672/21672), done.
linking server/addons to /var/tmp/openupgrade/11.0/addons
getting git://github.com/OpenUpgrade/OpenUpgrade.git
Cloning into ‘/var/tmp/openupgrade/11.0/server’…
remote: Counting objects: 25432, done.
remote: Compressing objects: 100% (20638/20638), done.
remote: Total 25432 (delta 7904), reused 10351 (delta 3820), pack-reused 0
Receiving objects: 100% (25432/25432), 119.52 MiB | 655.00 KiB/s, done.
Resolving deltas: 100% (7904/7904), done.
Checking connectivity… done.
Checking out files: 100% (22678/22678), done.
From https://github.com/OCA/openupgradelib
* branch master -> FETCH_HEAD
Already up-to-date.
copying database bes to mydb_migrated…
Copying the database using ‘with template’
running migration for 10.0
running migration for 11.0

—End—

See here too, https://doc.therp.nl/openupgrade/

Hope it helps,
Terry


  Save
ABOUT THE AUTHOR

TAF