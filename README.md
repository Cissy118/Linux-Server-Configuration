# Linux-Server-Configuration

### Basic Information
1. Server IP Address: 52.38.97.158 
2. SSH Port: 2200 
3. Public URL: ec2-52-38-97-158.us-west-2.compute.amazonaws.com <br>

### Steps
##### 1. Launch VM on AWS with Udacity account. 
Reference: https://www.udacity.com/account#!/development_environment
##### 2. SSH into your server as the root user. 
```
$ ssh -i ~/.ssh/udacity_key.rsa root@xx.xx.xx.xxx
```
Note: xx.xx.xx.xxx is server's ip address
##### 3. Create a user named `grader`.
Reference: https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04

* Login as root, create a new user named grader
```
@root $ adduser grader
```
* Switch to grader user
```
@root $ su - grader
```
* Copy the root’s public key into grader’s so that you can login as grader from your local host(where you have your private key)
```
@grader $ mkdir .ssh
@grader $ touch .ssh/authorized_keys
@root $ cat .ssh/authorized_keys
(copy entire file)
@grader $ nano .ssh/authorized_keys
(paste)
```
Note: use 'exit' to get back to root user
* Setup file permission on authorized_key file and ssh directory
```
@grader $ chmod 700 .ssh
@grader $ chmod 644 .ssh/authorized_keys
```
* Logout root user and test to login as grader user from local
```
@local  $ ssh -i ~/.ssh/udacity_key.rsa grader@xx.xx.xx.xxx
```
##### 4. Give the `grader` sudo permission.
Reference: https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps
* Add the grader to sudo group to ger sudo permission
```
@root $ gpasswd -a grader sudo
```
##### 5. Update all current installed the packages.
Reference: Udacity Course
```
$ sudo apt-get update
$ sudo apt-get upgrade
```
##### 6. Change the SSH port from 22 to 2200
Reference: https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-12-04
* Find the config file and edit
```
@grader $ nano /etc/ssh/sshd_config
```
* Change the port number 22 to 2200
* Change PermitRootLogin from `without-password` to `no`
```
@root $ serivce ssh restart
```
* Test if it works
```
@local $ ssh -i ~/.ssh/udacity_key.rsa root@xx.xx.xx.xxx -p 2200
```

##### 7. Configure the Uncomplicated Firewall (UFW).
Reference: Udacity Course
* Only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
```
@grader $ sudo default deny incoming
@grader $ sudo default allow outgoing
@grader $ sudo ufw allow 2200
@grader $ sudo ufw allow http
@grader $ sudo ufw allow ntp
@grader $ sudo ufw enable
```
* Check status
```
@grader $ sudo ufw status
```
##### 8. Configure the local timezone to UTC.
* Test if already UTC
```
$ date
Mon Mar 28 12:34:56 UTC 2016
```
If not, see reference: https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29

##### 9. Install and configure Apache to serve a Python mod_wsgi application.
Reference: http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html

@grader $ sudo apt-get install apache2
@grader $ sudo apt-get install python-setuptools libapache2-mod-wsgi
@grader $ sudo service apache2 restart

##### 10. Install git and deploy flask app on ubuntu vps
Reference: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
- Install and configure Git:
```
$ sudo apt-get install git
$ git config --global user.name "YOUR NAME"
$ git config --global user.email "YOUR EMAIL ADDRESS"
```
- Setup for deploying a Flask Application on Ubuntu VPS
```
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ sudo a2enmod wsgi
$ cd /var/www
  - $ sudo mkdir catalog
  - $ cd catalog and $ sudo mkdir catalog
  - $ cd catalog and $ sudo mkdir static templates
$ sudo nano __init__.py
```
- Paste in the following code:
```
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello, I love Digital Ocean!"
if __name__ == "__main__":
    app.run()
```
- Install Flask
```
$ sudo apt-get install python-pip
$ sudo pip install virtualenv
$ sudo virtualenv venv
$ sudo chmod -R 777 venv
$ source venv/bin/activate
$ pip install Flask
$ python __init__.py
$ deactivate
$ sudo nano /etc/apache2/sites-available/catalog.conf
```
* Paste in the following lines of code and change names and addresses regarding your application:
```
<VirtualHost *:80>
    ServerName PUBLIC-IP-ADDRESS
    ServerAdmin admin@PUBLIC-IP-ADDRESS
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
Enable the virtual host:
```
$ sudo a2ensite catalog
```
- Create the .wsgi File and Restart Apache
```
$ cd /var/www/catalog and $ sudo vim catalog.wsgi
```
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
* Restart Apache:
```
$ sudo service apache2 restart
```
* Clone GitHub repository and make it web inaccessible
```
$ git clone https://github.com/yourrepository
```
- Move all content of created Catalog-Web-App directory to /var/www/catalog/catalog/-directory and delete the leftover empty directory.
- Make the GitHub repository inaccessible:
```
$ cd /var/www/catalog/ and $ sudo vim .htaccess
```
  - Paste in the following:
  - RedirectMatch 404 /\.git

##### 11. Install and configure PostgreSQL.
Reference: - https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
- http://killtheyak.com/use-postgresql-with-django-flask/

@grader $ sudo apt-get install postgresql



remove or disable available sites
http://ubuntuforums.org/showthread.php?t=1692000


