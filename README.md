# Linux-Server-Configuration
Udacity Project 5 - Linux Server Configuration project

##Project Description

Installation of Ubuntu Linux on Amazon’s Elastic Compute Cloud (EC2) virtual machine to host  the Flask web application from Project 3: Item Catalog and make it accessible to the public.

The application is live at: [52.10.104.69][1] and [ec2-52-10-104-69.us-west-2.compute.amazonaws.com/][2]

## Summary of software installed and configurations made

#### Step 1. Launch virtual Machine with  Udacity account and launch and log in:
 -  `mv ~/Downloads/udacity_key.rsa ~/.ssh/`
 -  `chmod 600 ~/.ssh/udacity_key.rsa`
 -  `ssh -i ~/.ssh/udacity_key.rsa root@52.10.104.69`

#### Step 2: Create a new user named grader and grant this user sudo permissions.
 -  `adduser grader`
 -  `visudo` ( add "grader ALL=(ALL:ALL) ALL" under line "root ALL ..." )
 
#### Step 3. Update all currently installed packages:
 - `apt-get update`
 - `sudo apt-get upgrade`
 - `apt-get install unattended-upgrades`
 - `dpkg-reconfigure -plow unattended-upgrades`
 
#### Step 4. Configure the local timezone to UTC.
Sources: [link][11] | [link][12]
 - `dpkg-reconfigure tzdata`
 - `apt-get install ntp`
 - `vim /etc/ntp.conf` 
 
#### Step 5. Change the SSH port from 22 to 2200
Sources: [link][15]
 - `vim /etc/ssh/sshd_config` 
 - Change `PermitRootLogin` from `without-password` to `no`
 - Temporalily change PasswordAuthentication from `no` to `yes.`
 - Append `AllowUsers grader`
 - `service ssh restart`
 - Generate ssh key On local machine: `ssh-keygen`
 - Create ssh key and save in a created file called `grader_key` in the `.ssh` directory when prompted for a location
 - Copy saved key (public) from local machine into the ubuntu server 
 - `scp ~/.ssh/grader_key.pub grader@52.10.104.69`
 - Back on root session on the server, switch to grader: `su - grader`
 - `mkdir ~/.ssh`
 - `chmod 700 ~/.ssh`
 - `cat ~/grader_key.pub >> ~/.ssh/authorized_keys`
 - `rm ~/grader_key.pub`
 - `chmod 600 ~/.ssh/authorized_keys`
 - (if facing difficulties in previous steps, copy key manually into   .ssh/authorized_keys)
 - `exit`
 - `service ssh restart`

#### Step 6. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
Sources: [link][16] | [link][17]
 - `sudo ufw allow 2200/tcp`
 - `sudo ufw allow 80/tcp` 
 - `sudo ufw allow 123/udp`
 - `sudo ufw enable`
 
#### Step 7. Install and configure Apache to serve a Python mod_wsgi application 
 Sources: [link][5] | [link][6]
 - `apt-get install apache2`
 - `apt-get install python-setuptools libapache2-mod-wsgi`
 - `echo "ServerName ec2-52-10-104-69.us-west-2.compute.amazonaws.com" | sudo tee /etc/apache2/conf-available/fqdn.conf`
 - `a2enconf fqdn`
 - `apt-get install libapache2-mod-wsgi python-dev`
 - `a2enmod wsgi`
 - `service apache2 restart`
 
#### Step 8. Install and configure PostgreSQL:
Sources: [link][10]
 - `sudo apt-get install postgresql postgresql-contrib`
 - a) Do not allow remote connections (default):
 - `sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
 - Open the database setup file:
 - `sudo vim database_setup.py`
 - Change the line starting with "engine" to (fill in a password):
 - `python engine = create_engine('postgresql://catalog:catalogpsswd@localhost/catalog')`
 - Later change the same line in application.py and load_catalog_data.py once the git clone step is complete

 - b) Create a new user named catalog that has limited permissions to your catalog application database:
 - `sudo adduser catalog (password -> catalog)`
 - Change to default user postgres and connect to the db:
 - `sudo -u postgres psql postgres`
 - Create user with LOGIN role and set a password:
 - `CREATE USER catalog WITH PASSWORD 'catalogpsswd';`
 - Allow the user to create database tables:
 - `ALTER USER catalog CREATEDB;`
 - Create database:
 - `CREATE DATABASE catalog WITH OWNER catalog;`
 - Connect to the database catalog # \c catalog
 - Revoke all rights:
 - `REVOKE ALL ON SCHEMA public FROM public;`
 - Grant only access to the catalog role:
 - `GRANT ALL ON SCHEMA public TO catalog;`
 - `Exit out of PostgreSQl and the postgres user:`
 - `\q, then  exit`
 - Create postgreSQL database schema:
 - `python database_setup.py`
 - Load the initial data:
 - `python load_catalog_data.py` 
 
#### Step 9. Install git, clone and set up your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!
Sources: [link][18]
 - Install Git:
 - `sudo apt-get install git`
 - Set your name, e.g. for the commits:
 - `git config --global user.name "YOUR NAME"`
 - Set up your email address to connect your commits to your account:
 - `git config --global user.email "YOUR EMAIL ADDRESS"`
 - Clone project 3 repository on GitHub:
 - `git clone https://github.com/franzparks/Item-Catalog.git`
 - Move all content of created Item-Catalog directory to /var/www/catalog/catalog/ -directory and delete the empty directory.
 - `cd Item-Catalog`
 - `rm README.md`
 - `mv application.py __init__.py`
 - `mv * ..`
 - `cd ..`
 - `rm -rf Item-Catalog`

 - Make the GitHub repository inaccessible:
 
 - Create and open .htaccess file:
 - `cd /var/www/catalog/ and $ sudo vim .htaccess`
 - Paste in the following:
 - `RedirectMatch 404 /\.git`
 
 - Install python packages:
 - `apt-get install python-pip`
 - `pip install virtualenv`
 - `virtualenv venv`
 - `chmod -R 777 venv`
 - `pip install Flask-Seasurf`
 - `pip install requests`
 - `pip install --upgrade oauth2client`
 - `pip install sqlalchemy`
 - `pip install Flask-sqlalchemy`
 - `apt-get install python-psycopg2`
 - `pip install bleach`
 - `pip install dicttoxml`
 - `pip install dict2xml`

#### More configurations:
Sources: [link][7] | [link][8]

 - Extend Python with additional packages that enable Apache to serve Flask applications:
 - `sudo apt-get install libapache2-mod-wsgi python-dev`
 - Enable mod_wsgi (if not already enabled):
 - `sudo a2enmod wsgi`
 
##### Configure and Enable a New Virtual Host#
 - Create a virtual host config file
 - `sudo nano /etc/apache2/sites-available/catalog.conf`
 - Paste in the following lines of code and change names and addresses:
  ```
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
```
 - Enable the virtual host:
 - `sudo a2ensite catalog`
 
#### Create the .wsgi File and Restart Apache

 - Create wsgi file:
 - `cd /var/www/catalog and $ sudo vim catalog.wsgi`
 - Paste in the following lines of code:
  ```
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/catalog/")

  from catalog import app as application
  application.secret_key = 'Add your secret key'
  ```
  
 - Restart Apache:
 - `sudo service apache2 restart`
 
 - Activate the virtual environment:
 - `source venv/bin/activate`
 - Install Flask inside the virtual environment:
 - `pip install Flask`
 - Run the app:
 - `python __init__.py`

 
 #### Step 10. Install Glances and failban for monitoring and preventing abusive use of the system
 Sources: [link][13] | [link][14]
 - `apt-get install python-pip build-essential python-dev`
 - `pip install Glances`
 - `apt-get install lm-sensors`
 - `pip install PySensors`
 - `apt-get install fail2ban`
 - `cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
 - `vim /etc/fail2ban/jail.local`
  - set bantime  = 1200
  - destemail = grader@localhost
 - `apt-get install sendmail iptables-persistent`
 - `service fail2ban restart`
 
#### To check glances output:
 - `glances`

[1]: http://52.10.104.69/
[2]: http://ec2-52-10-104-69.us-west-2.compute.amazonaws.com/
[3]: https://wiki.ubuntu.com/Security/Upgrades
[4]: https://help.ubuntu.com/lts/serverguide/automatic-updates.html
[5]: http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html
[6]: https://help.ubuntu.com/lts/serverguide/httpd.html
[7]: http://httpd.apache.org/docs/2.2/en/mod/core.html#virtualhost
[8]: https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps
[9]: https://help.ubuntu.com/community/PostgreSQL
[10]: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04
[11]: https://help.ubuntu.com/community/UbuntuTime
[12]: https://help.ubuntu.com/lts/serverguide/NTP.html
[13]: https://pypi.python.org/pypi/Glances
[14]: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-fail2ban-on-ubuntu-14-04
[15]: https://wiki.archlinux.org/index.php/SSH_keys
[16]: https://help.ubuntu.com/community/UFW
[17]: https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server
[18]: https://help.github.com/articles/set-up-git/#platform-linux
