# Linux-server-configuration

In this project we will deploy an exiting application to linux server and configure it

Final url for application is http://ec2-18-185-99-217.eu-central-1.compute.amazonaws.com
`SSH PORT: 2200`


## Instructions for ssh access on local machine

1. Download Private Key from the on Amazon Lightsail.
2. Move the private key file into the folder `~/.ssh` (where ~ is your environment's home directory). 
3. Specify appropr# Linux-server-configuration

In this project we will deploy an exiting application to linux server and configure it

Final url for application is http://ec2-18-185-99-217.eu-central-1.compute.amazonaws.com
`SSH PORT: 2200`


## Instructions for ssh access on local machine

1. Download Private Key from the on Amazon Lightsail.
2. Move the private key file into the folder `~/.ssh` (where ~ is your environment's home directory). 
3. Specify appropriate permissions for the the directory and file using chmod
4. In your terminal, type in
	```ssh -i ~/.ssh/Lightsail-key.pem ubunut@address```

## Create user grader

1. `sudo adduser grader`
2. `sudo nano /etc/sudoers`
3. `sudo touch /etc/sudoers.d/grader`
4. `sudo nano /etc/sudoers.d/grader` 
5.  Type this in `grader ALL=(ALL:ALL) NOPASSWD:ALL`

## Set ssh login using keys

1. Generate keys on local machine using`ssh-keygen`

	On you virtual machine:
	```
	$ su - grader
	$ mkdir .ssh
	$ touch .ssh/authorized_keys
	$ vi .ssh/authorized_keys
	```
	Copy the public key (_one with the extension .pub_) 
  
	```
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys
	```
	
3. Restart SSH using `service ssh restart`
4. Now Login through ssh

	`ssh -i ~/.ssh/yourkeyfile grader@35.164.53.24`

## Update installed packages

	sudo apt-get update
	sudo apt-get upgrade

## Change the SSH port from 22 to 2200

1. Use `sudo vi /etc/ssh/sshd_config` and then change Port 22 to Port 2200 , save & quit.
2. Restart SSH using `sudo service ssh restart` 

## Configure Firewall (UFW)

Configure Firewall to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
`
sudo ufw allow ssh`
    `sudo ufw allow www`
`sudo ufw allow ntp`
  `sudo ufw allow 2200/tcp`
	`sudo ufw allow 80/tcp`
	`sudo ufw allow 123/udp`
	`sudo ufw enable` 
  `sudo ufw status`
  `
 
## Configure timezone to UTC

1. Configure time zone using `sudo dpkg-reconfigure tzdata`


## Install Apache to serve a Python mod_wsgi application

1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`

## Install and configure PostgreSQL

1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo vi /etc/postgresql/9.3/main/pg_hba.conf`
3. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`
5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell
	
	```
	CREATE DATABASE catalog;
  CREATE USER catalog;
	```
5. Set a password for user catalog
	
	```
	ALTER ROLE catalog WITH PASSWORD 'password';
	```
6. Give user "catalog" permission to "catalog" application database
	
	```
	GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
	
	```
	exit
	```
 
## Install git, clone and setup your Catalog App project.
1. Install Git using `sudo apt-get install git`
2. cd into /var/www/
3. touch FlaskApp
4. Clone the Catalog App to the virtual machine `git clone https://github.com/yasirrazakhan/Item-Catalog-app.git`
5. Rename `project.py` to `__init__.py` using `sudo mv website.py __init__.py`, if `__init__.py` not present.
6. Edit `database_setup.py` and `lotsofmenus.py` to change `engine = create_engine('sqlite:///useritem.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
7. Install pip `sudo apt-get install python-pip`
8. Use pip to install dependencies -
	* `sudo pip install sqlalchemy flask-sqlalchemy psycopg2 bleach requests`
	* `sudo pip install flask packaging oauth2client redis passlib flask-httpauth`
9. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
10. Create database schema `sudo python database_setup.py`
11. Populate database `sudo python lotsofmenus.py`


## Configure and Enable a New Virtual Host
1. Create FlaskApp.conf to edit: `sudo vi /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following lines of code to the file to configure the virtual host. 
	
	```
	<VirtualHost *:80>
		ServerName yourpublicip
		ServerAdmin name@example.com
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/static
		<Directory /var/www/FlaskApp/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```
3. Enable the virtual host with the following command: `sudo a2ensite FlaskApp`

## Create the .wsgi File
1. Create the .wsgi File under /var/www/FlaskApp: 
	
	```
	cd /var/www/FlaskApp
	sudo vi flaskapp.wsgi 
	```
2. Add the following lines of code:
	
	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/FlaskApp/")

	from FlaskApp import app as application
	application.secret_key = 'super_secret_key'
	```
  Restart server using sudo service apache2 restart
  
## RESOURCES

UDACITY CLASSROOM FORUM

https://www.vultr.com/docs/how-to-configure-ufw-firewall-on-ubuntu-14-04
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
