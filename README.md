# Linux Server Configuration Project
This is the fifth project for "Full Stack Web Developer Nanodegree" on Udacity.

Project Overview
You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

To view the compiled project visit: http://54.175.131.109 or http://ec2-54-175-131-109.compute-1.amazonaws.com

PORT WAS CHANGED FROM 22 to 2200

Key file contents noted in project submission notes. Create a PRIVATE KEY file with this content and point to it when logging in.

Login using ssh -i ~/.ssh/<<PRIVATEKEYFILE>> grader@ec2-54-175-131-109.compute-1.amazonaws.com -p 2200


## CONFIGURATIONS MADE:

## grader user created
Utilized Udacity Cource and  https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart to learn how to su to the new user

1. Run $ sudo adduser grader to create a new user named grader
2. Create sudoers file directory with sudo nano /etc/sudoers.d/grader
3. insert grader ALL=(ALL:ALL) ALL

## Set ssh login using keys Reference: Udacity Course
1. generate keys on local machine using`ssh-keygen` ; then save the private key in `~/.ssh` on local machine
2. deploy public key on developement enviroment

	As root user:
	```
	$ su - grader
	$ mkdir .ssh
	$ touch .ssh/authorized_keys
	$ sudo nano .ssh/authorized_keys
	```
	Copy the public key generated on your local machine to this file and save
	```
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys
	```

3. reload SSH using `service ssh restart`
4. now login via ssh -i ~/.ssh/<<PRIVATEKEYFILE>> grader@ec2-54-175-131-109.compute-1.amazonaws.com -p 22
    ( I had not yet update the port to 2200 at this point)

## Ensure all currently installed packages are updated

	sudo apt-get update
	sudo apt-get upgrade

## Change the SSH port from 22 to 2200
1. Use `sudo nano /etc/ssh/sshd_config`  change Port 22 to Port 2200 here:
    # What ports, IPs and protocols we listen for
    Port 2200
2. Reload SSH
    `sudo service ssh restart`

## Uncomplicated Firewall (UFW) udpates

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

	sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp
	sudo ufw allow 123/udp
	sudo ufw enable

## Configure the local timezone to UTC
1. Configure the time zone
     `sudo dpkg-reconfigure tzdata`

## Install and configure Apache to serve a Python mod_wsgi application
    Reference:
    https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-16-04

    https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps


1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`

## Install and configure PostgreSQL
1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
3. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`
5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell

	```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
5. Set a password for user catalog

	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
	```
6. Give user "catalog" permission to "catalog" application database

	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
7. Quit postgreSQL `postgres=# \q`
8. Exit from user "postgres"

	```
	exit
	```

## Install git, clone and setup your Catalog App project.
1. Install Git using `sudo apt-get install git`
2. Use `cd /var/www` to move to the /var/www directory
3. Create the application directory `sudo mkdir FlaskApp`
4. Move inside this directory using `cd FlaskApp`
5. Clone the Catalog App to the virtual machine `git clone https://github.com/jasonelijah/itemcatalog.git`
6. Rename the project's name `sudo mv ./Item_Catalog_UDACITY ./FlaskApp`
7. Move to the inner FlaskApp directory using `cd FlaskApp`
8. Rename `website.py` to `__init__.py` using `sudo mv website.py __init__.py`
9. Edit `database_setup.py`, `website.py` and `functions_helper.py` and change `engine = create_engine('sqlite:///toyshop.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
10. Install pip `sudo apt-get install python-pip`
11. Use pip to install dependencies `sudo pip install -r requirements.txt`
12. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
13. Create database schema `sudo python database_setup.py`

## Configure and Enable a New Virtual Host
1. Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following lines of code to the file to configure the virtual host.

	```
	<VirtualHost *:80>
        ServerName 54.175.131.109
        ServerAlias ec2-54-175-131-109.compute-1.amazonaws.com
        ServerAdmin admin@54.175.131.109
        WSGIDaemonProcess FlaskApp python-path=/var/www/FlaskApp:/var/www/FlaskApp/venv/lib/python2.7/site-packages
        WSGIProcessGroup FlaskApp
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
	sudo nano flaskapp.wsgi
	```
2. Add the following lines of code to the flaskapp.wsgi file:

	```
    activate_this = '/var/www/FlaskApp/FlaskApp/venv/bin/activate_this.py'
    execfile(activate_this, dict(__file__=activate_this))

    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/FlaskApp")

    from FlaskApp import app as application
    application.secret_key = 'super_secret_key'

	```

## Restart Apache
1. Restart Apache `sudo service apache2 restart `

## Other References:
http://www.informit.com/articles/article.aspx?p=1670957&seqNum=3
http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/


