# Linux Server Configuration
A project to set up a Linux server. Item Catalog web application runs live on Amazon Lightsail secure web server.

The item catalog is a RESTful web application that displays an item catalog allowing the user to login and add, manage items they add. 
SSH Info & Application URL
Server IP Address: 3.121.230.235
SSH access port: 2200
SSH login username: grader
Application URL: http://iisheek.com/
RESTful API: JSON http://iisheek.com/json


## 1 -  SSH from local machine to the lightsail virtual server instance
 Download the AWS provided default private key to the local machine
 Change the private key file permission 
 chmod 600 LightsailDefaultKey.pem
 Logged in, ssh successful  ssh ubuntu@3.121.230.235 -p 2200 -i /c/Users/PC/.ssh/LightsailDefaultKey.pem

## 2 -  Updating the System		
	sudo apt-get update
	sudo apt-get upgrade

## 3 - Create user grader & Give grader the permission to sudo
 sudo adduser grader
 sudo touch /etc/sudoers.d/grader
 sudo ls /etc/sudoers.d
 sudo nano /etc/sudoers.d/grader
   . grader ALL=(ALL) NOPASSWD:ALL
   . Save changes(Ctrl+X, Ctrl+Y, Enter)

## 4 - Create an SSH key pair for grader using the ssh-keygen tool
 ssh-keygen 
 Named it linuxServerConfig.pub
 Save the private key in ~/.ssh on local machine
 On the server instance i.e virtual machine:
    
    su - grader
    mkdir .ssh
    touch .ssh/authorized_keys
    nano .ssh/authorized_keys

    Copy the public key content from local machine to this file and save, change the access level
    chmod 700 .ssh
    chmod 644 .ssh/authorized_keys
    

### 5 - Change the SSH port from 22 to 2200
    
	sudo nano /etc/ssh/sshd_config
	sudo service ssh restart
	
    sudo ufw allow 2200/tcp
    sudo ufw allow www
    sudo ufw allow 123/udp
    sudo ufw deny 22
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw enable
    sudo ufw status
    
 SSH into grader using the key on 2200 port 
 ssh grader@3.121.230.235 -p 2200 -i ~/.ssh/linuxServerConfig

## 6 - Prepare to deploy the project

### Install and configure Apache to serve a Python mod_wsgi application
 sudo apt-get install libapache2-mod-wsgi
sudo apt-get install apache2
* Once done http://3.121.230.235 serves the default apache page

### Install and configure PostgreSQL
  sudo apt-get install postgresql
* Test postgres user
    
    sudo su - postgres
    psql
    \q
    exit
    
* Do not allow remote connections  
   
    sudo ls /etc/postgresql/9.5/main
    sudo cat /etc/postgresql/9.5/main/pg_hba.conf
    sudo service apache2 restart
   

* Create a new database user named catalog that has limited permissions to catalog application database
    
    sudo su - postgres
    CREATE USER catalog WITH PASSWORD 'catalog';
    \du
    CREATE DATABASE catalog;
    \l
    

### 7 - Deploy the Item Catalog app from git and create the required set up on server
* Create the FlaskApp, cloning the git repository
    
    cd /var/www
    sudo mkdir FlaskApp
    cd FlaskApp
    sudo git clone https://github.com/qadari/item-catalog.git
    tree
    sudo apt install tree
    tree
    
* Edit the python files to reflect the correct postgresql database connection - username, password, DBname
    sudo nano project.py
    sudo nano database_setup.py.py
    change engine = create_engine('sqlite:///itemcatalog.db') to engine = create_engine('postgresql://catalog:catalog@localhost/catalog')


### Install Flask other required modules

    sudo apt-get install python-pip
    sudo pip install virtualenv
    sudo virtualenv venv
    source venv/bin/activate
    sudo pip install Flask
    pip install httplib2
    sudo apt-get install python3-oauth2client
    sudo apt-get install python3-requests
    sudo apt-get install python-requests
    sudo apt-get install  python3-sqlalchemy
    sudo apt-get python3-psycopg2/sudo apt-get install python3-psycopg2
    sudo python project.py

### Enable the FlaskApp

    sudo nano /etc/apache2/sites-available/FlaskApp.conf

	<VirtualHost *:80>
	ServerName 3.121.230.235
			ServerAlias www.catalogsomali.cf
			ServerAdmin mmckaarshe@gmail.com
			WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
			<Directory /var/www/FlaskApp/item-catalog/>
					Order allow,deny
					Allow from all
			</Directory>
			Alias /static /var/www/FlaskApp/item-catalog/static
			<Directory /var/www/FlaskApp/item-catalog/static/>
					Order allow,deny
					Allow from all
			</Directory>
			ErrorLog ${APACHE_LOG_DIR}/error.log
			LogLevel warn
			CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>

     sudo a2ensite FlaskApp

### Create the .wsgi File

cd /var/www/FlaskApp
sudo nano flaskapp.wsgi
Add the following lines of code to the flaskapp.wsgi file

    #!/usr/bin/python3
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/FlaskApp/item-catalog/")

    from FlaskApp import app as application
    application.secret_key = 'super_secret_key'

### Restart Apache
sudo service apache2 restart

### Add tables and data
 sudo python /var/www/FlaskApp/item-catalog/database_setup.py
 
### run project app
cd /var/www/FlaskApp/item-catalog/
sudo python project.py 
 
















