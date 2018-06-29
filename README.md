# Linux Server Configuration Project

this project take a baseline installation of a Linux server and prepare it to host a web applications. Secure server from a number of attack vectors, install and configure a database server.

You can visit http://ec2-54-191-209-195.us-west-2.compute.amazonaws.com/ for the website deployed.

## Launch Virtual Machine

  Amazon Lightsail for this project
  
## Instructions for SSH access to the instance
1 Download Private Key (Private key in : "Notes to Reviewer")

2 Move the private key file into the folder ~/.ssh (home directory).

3 Change permissions : chmod 600 ~/.ssh/private_key_file

4 How to connect to the instance

	Public IP Address
	   54.191.209.195
	   
	User : grader
	
        SSH command
	ssh -i ~/.ssh/private_key_file grader@54.191.209.195


## Create a new user named grader

1. `sudo adduser grader`
2. `nano /etc/sudoers`
3. `nano /etc/sudoers.d/90-cloud-init-users`, add `grader ALL=(ALL:ALL) NOPASSWD: ALL`

## Update all currently installed packages

	sudo apt-get update
	sudo apt-get upgrade

## Change the SSH port from 22 to 2200
1. Use `sudo nano /etc/ssh/sshd_config` and then change Port 22 to Port 2200
2. Reload SSH : `sudo service ssh restart`

## Configure the Firewall (UFW)

Configure UFW to only allow incoming connections for : SSH (port 2200), HTTP (port 80), and NTP (port 123)

	sudo ufw default deny incoming
	sudo ufw default allow outgoing
	sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp
	sudo ufw allow 123/udp
	sudo ufw enable 
	
` $ ufw status
Status: active
 
To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123                        ALLOW       Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6) 
80/tcp (v6)                ALLOW       Anywhere (v6) 
123 (v6)                   ALLOW       Anywhere (v6)`
 
## Configure the local timezone to UTC
Local timezone is already set to UTC.
`
$ date +%Z
UTC`

If timezone is different, reconfigure tzdata and set to UTC:

`sudo dpkg-reconfigure tzdata`

## Install and configure Apache to serve a Python mod_wsgi application

1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`

## Install and configure PostgreSQL
1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`
3. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`
5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell
	
	```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
6. Set password for user catalog
	
	```
	postgres=# ALTER USER catalog WITH PASSWORD 'catalog';
	```
7. Grant permission to "catalog" user to "catalog" application database
	
	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```

## Install git, clone and setup your Catalog App project.
1. Install Git using `sudo apt-get install git`
2. Use `cd /var/www` to move to the /var/www directory 
3. Create the application directory `sudo mkdir FlaskApp`
4. Move inside this directory using `cd FlaskApp`
5. Clone the Catalog App to the virtual machine `git clone https://yourcatalogproject.git`
6. Rename the project's name `sudo mv ./yourcatalogproject ./FlaskApp`
7. Move to the inner FlaskApp directory using `cd FlaskApp`
8. Edit all python files with `engine = create_engine('sqlite:///somedatabase.db')` and change to `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
9. Install package needed to install and run the catalog application.
        
`       sudo apt-get install python-dev
	sudo apt-get install python-psycopg2
	sudo apt-get install python-pip
	sudo pip install flask-uploads
	sudo pip install flask-seasurf
	sudo pip install httplib2`


10. Create database schema `python models.py`
11. Upload information to de database : `python insert_info.py`

## Configure and Enable a New Virtual Host
*(Reference: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

1. Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following lines of code to the file to configure the virtual host. 
	
	```
	<VirtualHost *:80>
		ServerName 
		ServerAdmin elizabeth.moreno.aguilera@gmail.com
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
4. Disable default host: `sudo a2dissite 000-default`

## Create the .wsgi File
1. Create the .wsgi File under /var/www/FlaskApp: 
	
	```
	cd /var/www/FlaskApp
	sudo nano flaskapp.wsgi 
	```
2. Add the following lines of code to the flaskapp.wsgi file:
	
	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/FlaskApp/")

	from FlaskApp import app as application
	application.secret_key = 'Add your secret key'
	```

## Restart Apache
Restart Apache `sudo service apache2 restart `
