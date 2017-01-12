# Linux Server Configuration

This project is a part of the Full Stack Web Developer Nanodegree through [Udacity](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004).

## About
Configuring a Linux server to host [Catalog project](https://github.com/eakmotion/Catalog)

## Server details
IP address: 35.160.113.98
SSH Port: 2200  
Username: grader  
URL site: <http://35.160.113.98/>  

## Installed Packages
Name | Description
-----|------------
apache2 | Web Server
libapache2-mod-wsgi | Tool that serves Python web apps from Apache server
postgresql | Relational database system
sqlalchemy | SQL toolkit and ORM for Python
Flask | Python web framework
oauth2client | Python library for accessing resources protected by OAuth 2.0
httplib2 | A comprehensive HTTP client library
requests | Python HTTP Requests for Humans
git | Version control tool  

# Configuration summary:
## User Management: create a user with the permission to sudo

1. Create a new user  with name 'grader':
  - Create a new user with name 'grader':   
    `$ adduser grader`  
  - Grant user permission to perform sudo command:  
    `$ visudo`  
  -  Add the new user: grader below `root ALL=(ALL:ALL) ALL` line and save it::  
`$ grader ALL=(ALL:ALL) ALL`  

2. Check the new user is present as root
  - You can list all the users present as root:  
 `$ cut -d: -f1 /etc/passwd`  

## Update all currently installed packages

1. Update all available packages:  
  ` $ sudo apt-get update`

2. Upgrade packages to newer versions:  
  `$ sudo apt-get upgrade`

## Change SSH port from 22 to 2200
1. Edit the file `/etc/ssh/sshd_config` and change the line of `Port 2`  to: `Port 2200`
2. Then restart the SSH service:  
  `$ sudo service ssh restart`
3. Now you can connect as `grader` from local machine:  
  `$ ssh grader@35.160.113.98 -p 2200`

## Configure UFW to only allow incoming connections
1. Block all incoming connections on all ports by default:  
  `$ sudo ufw default deny incoming`
2. Allow outgoing connection on all ports:  
  `$ sudo ufw default allow outgoing`
3. Allow incoming connection for SSH on port 2200, HTTP on port 80 and NTP on port 123:

  ```
  $ sudo ufw allow 2200/tcp
  $ sudo ufw allow www  
  $ sudo ufw allow ntp  
  ```
4. Enable the firewall:  
`$ sudo ufw enable`

## Change timezone to UTC
1. Configure the local timezone to UTC:  
  `$ sudo dpkg-reconfigure tzdata`
2. Select UTC

## Install and configure Apache to serve a Python mod_wsgi application

1. Install Apache2 web server:  
  `$ sudo apt-get install apache2`
2. Install mod_wsgi and python-setuptools for serving Python apps from Apache:  
  `$ sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache:  
  `$ sudo service apache2 restart`

## Install Git and Setup Environment for delopying Flask Application

1. Install Git:   
  `$ sudo apt-get install git`
2. Clone the Catalog app on Github:

  ```
  $ cd /var/www
  $ mkdir FlaskApp  
  $ cd FlaskApp
  $ sudo git clone https://github.com/eakmotion/Catalog.git catalog
  ```

## Setup process for delopying Flask application

1. Install python, pip , virtualenv (in /var/www/FlaskApp/catalog):  

  ```
  $ sudo apt-get install libapache2-mod-wsgi python-dev
  $ sudo apt-get install python-pip
  $ sudo pip install virtualenv
  ```
2. Enable virtualenv:  

  ```
  $ sudo virtualenv venv
  $ source venv/bin/activate
  ```
3. Install following nessary packages in virtual environment:

  ```
  $ sudo apt-get install python-setuptools
  $ sudo apt-get install python-psycopg2
  $ sudo pip install Flask
  $ sudo pip install oauth2client
  $ sudo pip install requests
  $ sudo pip install httplib2
  $ sudo pip install sqlalchemy
  ```
4. Restart apache:  
  `$ sudo service apache2 restart`

# Install and configure PostgreSQL

1. Install PostgreSQL database:   
  `$ sudo apt-get install postgresql postgresql-contrib`
2. Create linux user for psql:   
	`$ sudo adduser catalog`
3. Change to user postgres:   
	`$ sudo su - postgre`
4. Connect to Postgres:   
	`$ psql`
5. Create user and password for your app:   
	`$ CREATE USER catalog WITH PASSWORD 'yourpassword';`
6. Create a database:   
	`$ CREATE DATABASE catalog;`
7. Grant only access to the catalog role:  

  ```
  $ \c catalog
  $ REVOKE ALL ON SCHEMA public FROM public;
  $ GRANT ALL ON SCHEMA public TO catalog;
  ```  
8. Navigate to Catalog directory:   
	`$ cd /var/www/FlaskApp/catalog/`
9. Edit the catalog.py, database_setup.py and data_seeder.py files by change:   
`engine = create_engine('sqlite:///catalog.db')`  
to  
`engine = create_engine('postgresql://catalog:yourpassword@localhost/catalog')`
10. Run database script to setup data:  

  ```
  $ python database_setup.py
  $ python data_seeder.py
  ```

# Configure and Enable a Virtual Host

Apache uses the .wsgi file to serve the Flask app   

1. Navigate to  /var/www/FlaskApp directory and create a file named catalog.wsgi:

  ```
  cd /var/www/FlaskApp
  sudo nano catalog.wsgi
  ```
2. Add the following codes to the catalog.wsgi file:  

  ```
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/FlaskApp/catalog/")

  from catalog import app as application
  application.secret_key = 'supersecret'
  ```
3. Configure the virtual host to our domain:  
	`$ sudo nano /etc/apache2/sites-available/FlaskApp.conf`
4. Add the following codes to the FlaskApp.conf:  

  ```
  <VirtualHost *:80>
    ServerName 35.160.113.98
    ServerAdmin grader@35.160.113.98
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/>
      Require all granted
    </Directory>
    Alias /static /var/www/catalog/static
    <Directory /var/www/catalog/static/>
      Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```

# Run the app
1. Restart Apache with the following command:  
  ` $ sudo service apache2 restart `
2. You may see a message similar to the following:  
  `Could not reliably determine the VPS's fully qualified domain name, using 127.0.0.1 for ServerName `
Ignore that message its just a warning
3. Go to this link: http://35.160.113.98

# Third-party Resources
- [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
- [Udacity discussion forum](https://discussions.udacity.com/t/solved-target-wsgi-script-cannot-be-loaded-as-python-module/209080)
- [Stack Overflow](http://stackoverflow.com/questions/12728004/error-no-module-named-psycopg2-extensions/37717229)
