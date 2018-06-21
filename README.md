# Linux Server

Author: Cheuk Lau

Date: 6/20/2018

In this project, we deploy the Catalog Application using Apache2 running on Amazon Web Services. The website is located at http://52.89.190.76.xip.io. The private key to ssh into the server as is not provided here.

## Get your server
1. Obtain an account at https://lightsail.aws.amazon.com.
2. Create a new `OS` project using Ubuntu.
3. Download key and move it to `/home/.ssh`.
4. Log into the server `ssh -i key ubuntu@52.89.190.75`.

## Secure your server
1. Update the installed package with `sudo apt-get update` then `sudo apt-get upgrade`.
2. Change the SSH port from 22 to 2200 by `sudo vi /etc/ssh/sshd_config` and changing port 22 to 2200. Then type `sudo service ssh restart`.
3. Change the firewall permissions by `sudo ufw allow 2200/tcp`, `sudo ufw allow 80/tcp`, `sudo ufw allow 123/udp` and `sudo ufw enable`.
4. Make sure the ports on https://lightsail.aws.amazon.com reflect the port changes in step 3.

## Give grader access
1. Create a new user named grader by `sudo adduser grader`.
2. Give grader sudo permission by `sudo touch /etc/sudoers.d/grader` and adding `grader ALL=(ALL:ALL) ALL` into the `grader` file.
3. Create an key-pair `key` and `key.pub` on your local machine using `ssh-keygen`.
4. Copy the contents of `key.pub` into `/home/grader/.ssh/authorized_keys`.
5. Change the following access `chmod 700 .ssh` and `chmod 664 .ssh/authorized_keys`.
6. Restart ssh using `sudo service ssh restart`.
7. Re-log in as grader using `ssh -i grader@52.89.190.76 -p 2200`.


## Prepare to deploy your project
1. Change the timezone to UTC by `sudo dpkg-reconfigure tzdata` and selecting `None` then `UTC`.
2. Install Apache2 by `sudo apt-get install apache2`.
3. Install WSGI to connect Apache2 to the Flask App by `sudo apt-get install libapache2-mod-wsgi.`. 
4. Restart Apache2 by `sudo service apache2 restart`.
5. Install PostgreSQL by `sudo apt-get install postgresql`.
6. Log in as postgres by `sudo su - postgres` then type `psql`.
7. Create new database and user named catalog by `CREATE DATABASE catalog;` then `CREATE USER catalog`.
8. Set a password for catalog user by `ALTER ROLE catalog WITH PASSWORD 'password'`.
9. Give catalog user permission to access database by `GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog`.

## Deploy the Item Catalog project
1. Install Git by `sudo apt-get install git`.
2. Create and change directory into `/var/www/category/`.
3. Clone the Catalog Application by `git clone https://github.com/cheuklau/catalog_app.git catalog`.
4. Rename te project by `sudo mv project.py __init__.py`.
5. Change `create_engine('sqlite:///catalog.db')` to `create_engine('postgresql://catalog:password@localhost/catalog')` in `__init__.py`, `database_setup.py` and `populate_database.py`. 
6. Install pip by `sudo apt-get install python-pip`.
7. Install all the dependencies by `sudo pip install sqlalchemy flask-sqlalchemy psycopg2 requests flask oauth2client`.
8. Install psycopg2 by `sudo apt-get -qqy install postgresql python-psycopg2`.
9. Create the database by `sudo python database_setup.py`.
10. Populate the database by `sudo python populate_database.py`.
11. Create Flask configuration file `sudo vi /etc/apache2/sites-available/catalog.conf`.
12. Add the following lines to catalog.conf:
`<VirtualHost *:80>
	ServerName 52.89.190.76
	ServerAdmin chucklingchuck@gmail.com
	WSGIScriptAlias / /var/www/cataglog/catalog.wsgi
	<Directory /var/www/catalog/catalog/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/catalog/catalog/static
	<Directory /var/www/catalog/catalog/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>`
13. Start the virtual host using `sudo a2ensite catalog`.
14. Create the WSGI file using `sudo vi /var/www/catalog/catalog.wsgi`.
15. Add the following lines to catalog.wsgi:
`#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")
from catalog import app as application
application.secret_key = 'super_secret_key'`
16. Restart Apache2 using `sudo service apache2 restart`.

## Credits

This project is generated as part of Udacity's full-stack web development program.