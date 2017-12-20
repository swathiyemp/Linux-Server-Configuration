# Linux Server Configuration  


This project is about performing baseline installation of a Linux distribution on a virtual machine and preparing it to host Item Catalog Application with updates installed, securing it from a number of attack vectors and installing/configuring web and database servers. 


## Getting Started        
1. Create an account in amazon aws-- https://aws.amazon.com/.
2. Choose Launch a virtual Machine option and then choose Light sail Instance.
3. Choose  Platform Linux and select OS only and then choose Ubuntu.
4. Pick your Instance plan, name it and hit create.
5. Click on your instance and hit connect using ssh and download private key from account page.
6. Save it in the directory .ssh and rename it like Lightsail Default Private Key.pem


## Prerequisites
*      Download and install Git--https://git-scm.com/downloads
*      Have an application like Item Catalog ready.


## Installing  
*       Install git with default specifications.
*       Open git bash and then type ssh ubuntu@34.232.46.54 -p 22 -i ~/.ssh/Lightsail Default Private Key.pem.
*      You will be connected to your Instance.
*       Update the software with this command sudo apt-get update.
*       Then type sudo apt-get upgrade
*       Set the timezone to UTC by typing this in sudo timedatectl set-timezone UTC and then check it by sudo timedatectl status.


###### Setting Up Security
*      Set up the servers security by changing port with the following command sudo nano /etc/ssh/sshd_config
*      Update the port to 2200 and then save it.
*      Then restart the server by sudo service ssh restart
*      Configure the firewall by performing below steps.
1.     First check the firewall status by  sudo ufw status
1.     Then type sudo ufw default deny incoming to block all incoming requests
1.     Then type sudo ufw default allow outgoing  to allow all outgoing traffic
1.     Then type in sudo ufw allow 2200/tcp  # SSH
1.     sudo ufw allow 80/tcp  # HTTP
1.     sudo ufw allow 123/udp  # NTP
1.     Enable the firewall by sudo ufw enable.
1.     Check the status by sudo ufw status.
1.     In the networking tab of lightsail Instance perform the following steps.
1.     Update ssh with custom tcp 2200
1      Also add custom udp 123.
1.     The server gets disconnected.Login from git bash with the following command--ssh ubuntu@34.232.46.54 -p 2200 -i ~/.ssh/Lightsail Default Private Key.pem


###### Creating a new user Grader
* Type the following command to get started sudo adduser grader
* Type sudo visudo
* It opens /etc/sudoers file
* Look for the line root    ALL=(ALL:ALL) ALL and then type in grader    ALL=(ALL:ALL) ALL following that line and then save it.
* Create a key pair for user grader on your local machine by opening git bash and typing ssh-keygen
* Save the key by entering the path.
* Now switch back to your instance and then type in su - grader
* Create a new directory by typing in mkdir ~/.ssh
* Create a new file by typing in nano ~/.ssh/authorized_keys
* Copy and paste the key that you saved on your local machine.
* Give the permissions to the user by the following command chmod 700 ~/.ssh && chmod 600 ~/.ssh/*
* Open the file by the following command sudo nano /etc/ssh/sshd_config
* Look for the password authentication field and change it to no and then save it.


 ###### Install the Apache

1. Type in sudo apt-get install apache2 to install apache.
1. Confirm it by going here--http://34.232.46.54:80
1. Install the mod_wsgi by typing in sudo apt-get install libapache2-mod-wsgi-py
1. Create a new directory for storing our web application by typing in sudo mkdir /var/www/itemcatalog
1. Change the ownership by typing in sudo chown -R ubuntu /var/www/itemcatalog
1. Create a new file itemcatalog.wsgi by typing in nano /var/www/itemcatalog/itemcatalog.wsgi
1. Enter the following into the file and save it.
                def application(environ, start_response):
                 status = '200 OK'
                 output = b'Hello World!'


                 response_headers = [('Content-type', 'text/plain'),
                        ('Content-Length', str(len(output)))]
                 start_response(status, response_headers)


             return [output]
1.   Create the configuration file by typing sudo nano /etc/apache2/sites-available/itemcatalog.conf and tye in the following

<VirtualHost *:80>
    ServerName ec2-34-232-46-54.us-east-1a.compute.amazonaws.com
    ServerAlias 34.232.46.54
  
   ServerAdmin 34.232.46.54
  
   WSGIScriptAlias / /var/www/itemcatalog/itemcatalog.wsgi  
    <Directory /var/www/itemcatalog/itemcatalog/>
      Order allow,deny
      Allow from all
  </Directory>
  
   Alias /static /var/www/itemcatalog/itemcatalog/static
 
  <Directory /var/www/itemcatalog/itemcatalog/static/>
      Order allow,deny
      Allow from all
  </Directory>
  
   ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

1.   Enable the virtual host by typing in sudo a2ensite itemcatalog
1.   Restart the server by typing in sudo apache2ctl restart


###### Setting up PostgresSQL


* Install the database using the following command sudo apt-get install postgresql.
* Then type CREATE ROLE catalog WITH LOGIN PASSWORD 'topsecret';
* Then type CREATE DATABASE catalogdb;
* Type \q to exit


###### Cloning the repository


* Type cd /var/www/itemcatalog/
* Clone it by typing git clone --single-branch https://github.com/swathiyemp/itemcatalog --branch master itemcatalog


###### Setting up python files
* First install pip by sudo apt-get install python-pip
* Set up virtual environment by sudo pip install virtualenv
* Cd into  /var/www/itemcatalog/itemcatalog/
* Type in virtualenv -p python venv
* Activate it by source venv/bin/activate
* Then install the required files by pip install -r requirements.txt(The requirements.txt can be formed by typing pip freeze in your local machine and then collecting the list and storing them into requirements.txt in your project folder)
* Then run the python files by typing python itemscatalog.py and python listofitems.py
* Then type deactivate
* Edit the WSGI file by sudo nano /var/www/itemcatalog/itemcatalog.wsgi
and then paste these below.


import sys
sys.path.insert(0, '/var/www/itemcatalog/itemcatalog')

activate_this = '/var/www/itemcatalog/itemcatalog/venv/bin/activate_this.py'
with open(activate_this) as file_:
exec(file_.read(), dict(__file__=activate_this))

from itemcatalog import app as application

## Running The Application

* Type in Restart Apache sudo apache2ctl restart
* Goto http://34.232.46.54 in your browser to view your application.
