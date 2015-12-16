# Linux-Server-Configuration
Udacity Project 5 - Linux Server Configuration project
Project Description

Installation of Ubuntu Linux on Amazon’s Elastic Compute Cloud (EC2) virtual machine to host  the Flask web application from Project 3: Item Catalog and make it accessible to the public.

The application is live at: 52.10.104.69 and ec2-52-10-104-69.us-west-2.compute.amazonaws.com

##Steps to prepare the server

Step 1: 
Create new development environment at:
https://www.google.com/url?q=https://www.udacity.com/account%23!/development_environment&sa=D&usg=AFQjCNGv6S6CtUsCXf6gj9FkVtiDLeeq0A

- Download private key and move the private key file into the folder ~/.ssh 
$mv ~/Downloads/udacity_key.rsa ~/.ssh/
- Restrict the key file to this account only (read+write)
$chmod 600 ~/.ssh/udacity_key.rsa
-SSH into the instance
$ssh -i ~/.ssh/udacity_key.rsa root@52.10.104.69

Step 2:
- Create a new user named grader and grant this user sudo permissions.
adduser grader
-Give grader permission to sudo:
 - Open the sudo configuration:
  $ visudo
 -Add the following line below root ALL...:
  grader ALL=(ALL:ALL) ALL

Step 3:
-Update all currently installed packages:
$sudo apt-get update
$sudo apt-get upgrade
 -Install the unattended-upgrades package:
 $sudo apt-get install unattended-upgrades
 -Enable the unattended-upgrades package:
 $sudo dpkg-reconfigure -plow unattended-upgrades

Step 4:
-Configure the local timezone to UTC. 
 -Open the timezone selection dialog:
 $sudo dpkg-reconfigure tzdata
 -Choose 'None of the above'
 -Choose UTC
 - Install ntp for time synchronization
 $sudo apt-get install ntp

Step 5:
-Change the SSH port from 22 to 2200
 -Open the config file:
 $sudo vim /etc/ssh/sshd_config
 -Change port from 22 to 2200
 -Change PermitRootLogin from without-password to no
 -Temporalily change PasswordAuthentication from no to yes.
 -Append AllowUsers grader
 -Restart SSH Service:
 -sudo service ssh restart

Step 6:
-Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
$sudo ufw allow 2200/tcp
$sudo ufw allow 80/tcp
$sudo ufw allow 123/udp
$sudo ufw enable

Step 7:
-Install and configure Apache to serve a Python mod_wsgi application

-Install Apache web server:
$ sudo apt-get install apache2
Open the ip address http://52.10.104.69/ - in a browser and on top of the page it should say 'It works!'.
-Install mod_wsgi for serving Python apps from Apache and the helper package python-setuptools:
$sudo apt-get install python-setuptools libapache2-mod-wsgi
$echo "ServerName ec2-52-10-104-69.us-west-2.compute.amazonaws.com" | sudo tee /etc/apache2/conf-available/fqdn.conf
a2enconf fqdn
apt-get install libapache2-mod-wsgi python-dev
a2enmod wsgi
Restart the Apache server for mod_wsgi to load:
$sudo service apache2 restart


Step 8:
-Install and configure PostgreSQL:

-Install PostgreSQL:
$ sudo apt-get install postgresql postgresql-contrib
-a) Do not allow remote connections (default):
$ sudo vim /etc/postgresql/9.3/main/pg_hba.conf
-Open the database setup file:
$ sudo vim database_setup.py
-Change the line starting with "engine" to (fill in a password):
python engine = create_engine('postgresql://catalog:catalogpsswd@localhost/catalog')
-Change the same line in application.py and load_catalog_data.py------------
-Rename application.py:
$ mv application.py __init__.py------------------
-b) Create a new user named catalog that has limited permissions to your catalog application database:
$ sudo adduser catalog (password -> catalog)
-Change to default user postgres and connect to the db:
$sudo -u postgres psql postgres

-Create user with LOGIN role and set a password:
# CREATE USER catalog WITH PASSWORD 'catalogpsswd';
-Allow the user to create database tables:
# ALTER USER catalog CREATEDB;

-Create database:
# CREATE DATABASE catalog WITH OWNER catalog;
-Connect to the database catalog # \c catalog
Revoke all rights:
# REVOKE ALL ON SCHEMA public FROM public;
-Grant only access to the catalog role:
# GRANT ALL ON SCHEMA public TO catalog;
-Exit out of PostgreSQl and the postgres user:
# \q, then $ exit
-Create postgreSQL database schema:
$ python database_setup.py
-Load the initial data:
$python load_catalog_data.py

Step 9:
-Install git, clone and set up your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!

-Install Git:
$ sudo apt-get install git
-Set your name, e.g. for the commits:
$ git config --global user.name "YOUR NAME"
-Set up your email address to connect your commits to your account:
$ git config --global user.email "YOUR EMAIL ADDRESS"
-Clone project 3 repository on GitHub:
$ git clone https://github.com/franzparks/Item-Catalog.git
-Move all content of created Item-Catalog directory to /var/www/catalog/catalog/ -directory and delete the empty directory.

-Make the GitHub repository inaccessible:

-Create and open .htaccess file:
$ cd /var/www/catalog/ and $ sudo vim .htaccess
-Paste in the following:
RedirectMatch 404 /\.git

Install python packages:

apt-get install python-pip
pip install virtualenv
virtualenv venv
chmod -R 777 venv
pip install Flask-Seasurf
pip install requests
pip install --upgrade oauth2client
pip install sqlalchemy
pip install Flask-sqlalchemy
apt-get install python-psycopg2
pip install bleach
pip install dicttoxml
pip install dict2xml


More configurations:
Extend Python with additional packages that enable Apache to serve Flask applications:
$ sudo apt-get install libapache2-mod-wsgi python-dev
Enable mod_wsgi (if not already enabled):
$ sudo a2enmod wsgi
Create a Flask app:
Move to the /var/www/catalog/catalog directory:
$ cd /var/www/catalog/catalog
$ sudo mkdir static templates
Create the file that will contain the flask application logic:
$ sudo nano __init__.py
Paste in the following code:
  from flask import Flask  
  app = Flask(__name__)  
  @app.route("/")  
  def hello():  
    return "I am working!"  
  if __name__ == "__main__":  
    app.run()  

Activate the virtual environment:
$ source venv/bin/activate
Install Flask inside the virtual environment:
$ pip install Flask
Run the app:
$ python __init__.py
Deactivate the environment:
$ deactivate
Configure and Enable a New Virtual Host#
Create a virtual host config file
$ sudo nano /etc/apache2/sites-available/catalog.conf
Paste in the following lines of code and change names and addresses:
  <VirtualHost *:80>
      ServerName 52.10.104.69
      ServerAdmin admin@52.10.104.69
      ServerAlias ec2-52-10-104-69.us-west-2.compute.amazonaws.com
      WSGIScriptAlias / /var/www/catalog/catalog.wsgi
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
  </VirtualHost>
Enable the virtual host:
$ sudo a2ensite catalog
Create the .wsgi File and Restart Apache

Create wsgi file:
$ cd /var/www/catalog and $ sudo vim catalog.wsgi
Paste in the following lines of code:
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/catalog/")

  from catalog import app as application
  application.secret_key = 'Add your secret key'
Restart Apache:
$ sudo service apache2 restart

Step 10 and extra : (added hostname and ip address to google console for oauth configuration)

- Install Glances and failban for monitoring

-apt-get install python-pip build-essential python-dev
-pip install Glances
-apt-get install lm-sensors
-pip install PySensors
-apt-get install fail2ban
-cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
-vim /etc/fail2ban/jail.local
-set bantime = 1200
-destemail = grader@localhost
-apt-get install sendmail iptables-persistent
-service fail2ban restart

To run glances and see some output in the terminal:
$ glances
