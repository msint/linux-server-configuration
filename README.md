# Linux Server Configuration

* Author: May Wong
* Email: msg2may@gmail.com

## Project Description:

This project is for Udacity Full Stack Web Development Nano Degree program Linux Server Configuration.
This project takes a baseline installation of a Linux distribution on a virtual machine and prepare it to host the web applications. Then, secure the server from a number of attack vectors, install and configure a Postgresql database server, and deploy "Item Catalog" web application onto it.

## Server Info:

  * Server IP Address: 54.224.53.245
  * SSH port: 2200
  * Application URL: http://54.224.53.245

## Connect to AWS Lightsail Instance via SSH:

  * Download the private key LightsailDefaultKey-us-east-1.pem from AWS
  * Logon to remote VM as root user named "ubuntu" via SSH:
  ```
    ssh -i ~/.ssh/LightsailDefaultKey-us-east-1.pem ubuntu@54.224.53.245
  ```  

## Update all currently installed packages as "ubuntu" user:
  ```
    sudo apt-get update
    sudo apt-get upgrade
   ```  

## Add a new user "grader":
  ```
    sudo adduser grader
  ```
  * Give user name as "Udacity Grader" and password as "password"


## Grant sudo permission to user "grader":

  * Add a new file named "grader" under "/etc/sudoers.d/" directory
  ```
    sudo nano /etc/sudoers.d/grader
  ```
  * Type "grader ALL=(ALL:ALL) ALL" in the grader file, save and quit.

## Check that user "grader" has sudo permission:

    ```
      sudo cat /etc/passwd
    ```
    If the above command will show the contents of the file, grader user has sudo permission.

## Generate key for ssh login for user "grader":

  * On your local machine enter, ```ssh-keygen``` and save the private key in ~/.ssh/graderKey

  * On your virtual machine, switch the user to "grader" account and create .ssh directory and authorized_keys file in /home/grader/.ssh/authorized_keys:
  ```
    su - grader
    mkdir .ssh
    nano .ssh/authorized_keys
  ```
  * Copy the public key from local machine file named ".ssh/graderKey.pub" to virtual machine file named ".ssh/authorized_keys" and save it.

  * Change the permission for directory and file .ssh/authorized_keys
  ```
    chmod 700 .ssh
    chmod 644 .ssh/authorized_keys
  ```

  * Reload ssh and enter password:
  ```
    service ssh restart
  ```

## Login with ssh as user "grader" using key (default port is 22):

   ```
    ssh -i .ssh/graderKey grader@54.224.53.245
   ```

## Update all currently installed packages as "grader" user:
   ```
    sudo apt-get update
    sudo apt-get upgrade
   ```  

## Change SSH port from 22 to 2200 in the following file:
    ```
      sudo nano /etc/ssh/sshd_config
    ```
    - In the above file, change Port to 2200
    - Change PasswordAuthentication to no
    - Change PermitRootLogin to no
    - Save and quit the file.
    - Restart ssh service
    ```
      sudo service ssh restart
    ```
    - In AWS Lightsail Firewall Security, add 2200 as the inbound custom TCP Rule port.

## Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

    ```
      sudo ufw default deny incoming
      sudo ufw default allow outgoing
      sudo ufw allow 2200/tcp
      sudo ufw allow 80/tcp
      sudo ufw allow 123/udp
      sudo ufw enable
    ```

## Check the status of Firewall configuration:
    ```
      sudo ufw status
    ```
    Status: active

    To                         Action      From
    --                         ------      ----              
    2200/tcp                   ALLOW       Anywhere                  
    80/tcp                     ALLOW       Anywhere                  
    123/udp                    ALLOW       Anywhere                             
    2200/tcp (v6)              ALLOW       Anywhere (v6)             
    80/tcp (v6)                ALLOW       Anywhere (v6)             
    123/udp (v6)               ALLOW       Anywhere (v6)    


## Confirm login with key as user grader to port 2200:
  ```
    ssh grader@54.224.53.245 -p 2200 -i ~/.ssh/graderKey
  ```
  
## Configure the local timezone to UTC

  ```
    sudo dpkg-reconfigure tzdata
  ```
  Timezone is already set to UTC

## Install Apache:
  ```
    sudo apt-get install apache2
  ```

## Install mod_wsgi:
  ```
    sudo apt-get install python-setuptools libapache2-mod-wsgi
  ```

## Restart Apache:
  ```
    sudo service apache2 restart
  ```

## Install and configure PostgreSQL:

  - Install PostgreSQL ```sudo apt-get install postgresql```
  - Login as user "postgres" ```sudo su - postgres```
  - Get to PostgreSQL shell: ```psql```
  - Create a new database and user name both named "catalog":
    ```
      CREATE DATABASE catalog;
      CREATE USER catalog;
    ```
  - Set "catalog" user's password: ```ALTER ROLE catalog WITH PASSWORD 'password';```
  - Give "catalog" user permission to "catalog" application database
    ```
      GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
    ```
  - Quit and exit postgreSQL:
    ```
      \q
      exit
    ```

## Install git, clone the Item Catalog project and configure:
    ```
      sudo apt-get install git
      cd /var/www/
      sudo mkdir catalogApp
      sudo git clone https://github.com/msint/movie-item-catalog.git catalogApp
      cd catalogApp
    ```
  - Make sure .git directory is not publicly accessible: ```sudo chmod 700 /var/www/catalogApp/catalogApp/.git```
  - In /var/www/catalogApp/itemCatalogApp directory, change file name application.py to
  __init__.py
  - Inside __init__.py, database_setup.py, and populateCatalog.py, change
    ```
      engine = create_engine('sqlite:///MovieCatalog.db')
    ```
    to
    ```
      engine = create_engine('postgresql://catalog:password@localhost/catalog')
    ```
  - Create database schema: ```sudo python database_setup.py```
  - Populate and add data to database: ```sudo python populateCatalog.py```


## Install App dependencies:

  - sudo pip install Flask
  - sudo pip install sqlalchemy
  - sudo pip install Flask-SQLAlchemy
  - sudo pip install psycopg2
  - sudo apt-get install python-psycopg2
  - sudo pip install flask-seasurf
  - sudo pip install oauth2client
  - sudo pip install httplib2
  - sudo pip install requests


## Install virtual environment:

  - Install the virtual environment: ```sudo pip install virtualenv```
  - Create a new virtual environment: ```sudo virtualenv venv```
  - Activate the virutal environment: ```source venv/bin/activate```
  - Change permissions: ```sudo chmod -R 777 venv```


## Configure and Enable a New Virtual Host

  - Create catalogApp.conf file:
    ```
      sudo nano /etc/apache2/sites-available/catalogApp.conf

      <VirtualHost *:80>
	       ServerName 54.224.53.245
	       ServerAdmin msg2may@gmail.com
	       WSGIScriptAlias / /var/www/catalogApp/catalogApp.wsgi
	       <Directory /var/www/catalogApp/catalogApp/>
		         Order allow,deny
		         Allow from all
	       </Directory>
	       Alias /static /var/www/catalogApp/catalogApp/static
	       <Directory /var/www/catalogApp/catalogApp/static/>
		        Order allow,deny
		        Allow from all
	       </Directory>
	       ErrorLog ${APACHE_LOG_DIR}/error.log
	       LogLevel warn
	       CustomLog ${APACHE_LOG_DIR}/access.log combined
      </VirtualHost>
    ```
  - Enable the virtual host: ```sudo a2ensite catalogApp```


## Create .wsgi file:

    ```
      cd /var/www/catalogApp
      sudo nano catalogApp.wsgi
    ```
    - In catalogApp.wsgi file:
    ```
      #!/usr/bin/python
      import sys
      import logging
      logging.basicConfig(stream=sys.stderr)
      sys.path.insert(0,"/var/www/catalogApp/")

      from catalogApp import app as application
      application.secret_key = 'super_secret_key'
    ```

## Restart Apache and application is up and running:

    ```
      sudo service apache2 restart
    ```
    - Application is up and running in http://54.224.53.245


## Note:

  * Facebook OAuth is currently enforcing to use https and it's not working at this moment.
   
   
## Acknowledgments
  * https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
