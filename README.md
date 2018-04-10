# Linux Server Configuration project
deploying a flask application on to AWS Light Sail.

IP address : 52.28.75.112

SSH port :2200


## summary


## Updating the server

```
sudo apt-get update
sudo apt-get upgrade
```

## Creating the New user with sudo permissions
 create a new user called grader:
```
sudo adduser grader
```
After the user gets created, run this command to give the grader sudo priviledges:
```
sudo visudo
```
Once your in the visudo file, go to the section user privilege specification
```
root    ALL=(ALL:ALL) ALL
```
You've noticed the root user in the file. You need add the grader user to same section.  It should look like this:
```
grader  ALL=(ALL:ALL) ALL
```
If you want to login into the account, check to see if works: `sudo login grader`.
We've created our new user, lets configure our ssh to non-default port

## Configuring SSH to a non-default port

SSH default port is 22. We want to configure it to non default port.  Since we are using LightSail, we need to open the non default port through the web interface. If you don't do this step, you'll be locked out of the instance. Go to the networking tab on the management console. Then, the firewall option


Go into your sshd config file: `sudo nano /etc/ssh/sshd_config`
Change the following options. # means commented out

```
# What ports, IPs and protocols we listen for
# Port 22
Port 2200

# Authentication:
LoginGraceTime 120
#PermitRootLogin prohibit-password
PermitRootLogin no
StrictModes yes

# Change to no to disable tunnelled clear text passwords
PasswordAuthentication No
```
Once these changes are made, restart ssh: `sudo service restart ssh`
### Creating the ssh keys
```
LOCAL MACHINE:~$ ssh-keygen
```

Once the keys are generated, you'll need to login to the user account aka grader here:
You'll make a directory for ssh and store the public key in an authorized_keys files

```
mkdir .ssh
cd .ssh
```
When you are in the directory, create an authorized_keys file. This where you paste the public key that you generated on your local machine.
```
sudo nano authorized_keys
```
Please double check your path by pwd. You should be in your `/home/grader/.ssh/` when creating the file. You should be login with grader account using the private key from local machine into the server: `ssh user@PublicIP -i ~/.ssh/whateverfile_id_rsa`

## Configuring Firewall rules using UFW
We need to configure firewall rules using UFW. Check to see if ufw is active: `sudo ufw status`. If not active, lets add some rules
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200
sudo ufw allow 80
sudo ufw allow 123
```
Now, you enable the rules: `sudo ufw enable` and re check the status to see what rules are activity

## Configure timezone for server
```
sudo dpkg-reconfigure tzdata
```
Choose none of the above and choose UTC.  The server by default is on UTC.

## Install Apache, Git, and flask

### Apache
We will be installing apache on our server. To do that:

```
sudo apt-get install apache2
```
If apache was setup correctly, a welcome page will come up when you use the PublicIP. We are going to install mod wsgi, python setup tools, and python-dev
```
sudo apt-get install libapache2-mod-wsgi python-dev
```
We need to enable mod wsgi if it isn't enabled: `sudo a2enmod wsgi`

Let's setup wsgi file and sites-available conf file for our application.
Create the WSGI file in `path/to/the/application directory`
```
WSGI file

import sys
sys.path.insert(0, '/var/www/itemsCatalog/vagrant/catalog')

from application import app as application
application.secret_key='super_secret_key'
```

Please note application.py is my python file. Wherever you housed your application logic.

in file `/etc/apache2/sites-enabled/000-default.conf `:

add the following line at the end of the <VirtualHost *:80> block right before the closing </VirtualHost>:
```
  WSGIScriptAlias / /var/www/myapp.wsgi
```
### Git
`sudo apt-get install git`

Clone repository into the apache directory. In the `cd /var/www`

```
sudo git clone https://github.com/rawda95/Item-Catalog.git
```

### Create database
`sudo su - postgres`
Type in `psql` as postgres user

Do following the commands:
```
postgres=# CREATE USER catalog WITH PASSWORD 'catalog';
postgres=# ALTER USER catalog CREATEDB;
postgres=# CREATE DATABASE catalog WITH OWNER catalog;
```
Connect to the catalog database: `\c itemscatalog`
```
itemscatalog=# REVOKE ALL ON SCHEMA public FROM public;
itemscatalog=# GRANT ALL ON SCHEMA public TO catalog;
```
Exit postgres and postgres user
```
postgres=# \q
postgres@PublicIP~$ exit
```

We need to update our database setup and application python files to illustrate the new engine connection
`engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`


##  third-party resources
 python

 Apache2

 Firewall (UFW)

 Python mod_wsgi

 ssh-keygen

 AWS-server instance

 Git

 PostgreSQL

 Flask