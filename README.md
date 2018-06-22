# Linux Server

Author: Cheuk Lau

Date: 6/21/2018

In this project, we deploy the Catalog Application (Python Flask framework with an SQL backend) using Apache2 running on Amazon Web Services. The website is located at http://34.220.16.149.xip.io. To SSH into the server as the grader type `ssh -i /path/to/key_grader -p 2200 grader@34.220.16.149` into the terminal. The private key `key_grader` is not publicly available.

## Get your server
1. Obtain an account at https://lightsail.aws.amazon.com.
2. Create a new OS project using Ubuntu.
3. Download `amazon_key` and move it to `/home/.ssh`.
4. Log into the server using `ssh -i /home/.ssh/amazon_key ubuntu@34.220.16.149`.

## Secure your server
1. Update the installed packages.

	`sudo apt-get update`

	`sudo apt-get upgrade`

2. Change the SSH port in `sshd_config` from 22 to 2200.

	`sudo vi /etc/ssh/sshd_config`

	`sudo service ssh restart`

3. Change firewall permissions.

	`sudo ufw allow 2200/tcp`

	`sudo ufw allow 80/tcp`

	`sudo ufw allow 123/udp`

	`sudo ufw enable`

4. Make sure ports on https://lightsail.aws.amazon.com reflect changes in the last step.

## Give grader access
1. Create a new user named grader. 

	`sudo adduser grader`

2. Give grader sudo permission.

	`sudo vi /etc/sudoers`

   Add the following line to `sudoers`.

	`grader ALL=(ALL:ALL) ALL` 

3. Create a key-pair `key_grader` and `key_grader.pub` in `/home/.ssh` on your local machine.

	`ssh-keygen`

4. Copy the contents of `key_grader.pub` on local machine into `/home/grader/.ssh/authorized_keys`.
5. Perform permission changes. 

	`chmod 700 .ssh`

	`chmod 664 .ssh/authorized_keys`

6. Restart ssh.

	`sudo service ssh restart`

7. Log in as grader. 

	`ssh -i /home/.ssh/key_grader grader@34.220.16.149 -p 2200`

## Prepare to deploy your project
1. Change the timezone to UTC.

	`sudo dpkg-reconfigure tzdata`

2. Install Apache2.

	`sudo apt-get install apache2`.

3. Install WSGI to connect Apache2 to the Flask App.

	`sudo apt-get install libapache2-mod-wsgi.`

4. Restart Apache2.

	`sudo service apache2 restart`

## Setting up the database
1. Install postgresql.

	`sudo apt-get install postgresql`

2. Log in as postgres.

	`sudo su - postgres`

3. Start postgresql.

	`psql`

7. Create new database and user named catalog.

	`CREATE DATABASE catalog;`

	`CREATE USER catalog;`

8. Set a password for catalog user. 

	`ALTER ROLE catalog WITH PASSWORD 'password'`

9. Give catalog user permission to access database. 

	`GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog`

## Deploy the Item Catalog project
1. Install Git.

	`sudo apt-get install git`

2. Create and change directory into `/var/www/FlaskApp/`.
3. Clone the Item Catalog project.

	`git clone https://github.com/cheuklau/catalog_app.git FlaskApp`.

4. Rename the project file.

	`sudo mv project.py __init__.py`.

5. Change `create_engine('sqlite:///catalog.db') to create_engine('postgresql://catalog:password@localhost/catalog')`in `__init__.py`, `database_setup.py` and `populate_database.py`.
6. Install pip.

	`sudo apt-get install python-pip`.

7. Install all the dependencies.

	`sudo pip install sqlalchemy flask-sqlalchemy psycopg2 requests flask oauth2client`.

8. Install psycopg2.

	`sudo apt-get -qqy install postgresql python-psycopg2`.

9. Create the database.

	`sudo python database_setup.py`

10. Populate the database.

	`sudo python populate_database.py`

11. Create Flask configuration file `sudo vi /etc/apache2/sites-available/FlaskApp.conf`.
12. Add the following lines to catalog.conf:

```
<VirtualHost *:80>  
	ServerName 34.220.16.149  
	ServerAdmin chucklingchuck@gmail.com  
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

13. Start the virtual host.

	`sudo a2ensite catalog`

14. Create the WSGI file.

	`sudo vi /var/www/catalog/catalog.wsgi`

15. Add the following lines to `catalog.wsgi`.

```
#!/usr/bin/python  
import sys  
import logging  
logging.basicConfig(stream=sys.stderr)  
sys.path.insert(0,"/var/www/FlaskApp/")  
from FlaskApp import app as application  
application.secret_key = 'super_secret_key'
```

16. Restart Apache2.

	`sudo service apache2 restart`

## Getting Google Authentication to work
1. Update `client_secrets.json` if using a new Google App.
2. Change `client_secrets.json` path in `init.py` to the full path `/var/www/catalog/catalog/client_secrets.json`.
3. Update `client_id` in `__init.py` if using a new Google App.
4. Change `app.run(host=*, port=*)` to `app.run()`.
5. Change gdisconnet link paths to `http://34.220.16.149.xip.io`.
6. Update  `data-clientid` in `static/login.html` if using a new Google App.
7. Update Authorized Javascript origins to include:

	`http://34.220.16.149.xip.io`

	`http://ec2-34-220-16-149.compute-1.amazonaws.com`

   and Authorized redirect URIs to include:

   	`http://ec2-34-220-16-149.compute-1.amazonaws.com/login`

   	`http://ec2-34-220-16-149.compute-1.amazonaws.com/gconnect`

   	`http://ec2-34-220-16-149.compute-1.amazonaws.com/oauth2callback`

## Credits

This project is generated as part of Udacity's full-stack web development program.