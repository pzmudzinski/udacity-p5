# udacity-p5
Linux Server Configuration

## TASKS

1. Launch your Virtual Machine with your Udacity account
2. Follow the instructions provided to SSH into your server
3. Create a new user named grader
4. Give the grader the permission to sudo
5. Update all currently installed packages
6. Change the SSH port from 22 to 2200
7. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
8. Configure the local timezone to UTC
9. Install and configure Apache to serve a Python mod_wsgi application
10. Install and configure PostgreSQL:  
 10.1 Do not allow remote connections  
 10.2 Create a new user named catalog that has limited permissions to your catalog application database
11. Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!

## IP ADRESSS
52.26.76.149

## WEB APP ADDRESS
http://52.26.76.149/catalog/

## SSH PORT
2200 

## COMMANDS
```
# adding new user  (3.)
> adduser grader

# adding root privileges (4.)
> gpasswd -a grader sudo 

#on localhost
> pbcopy <  ~/.ssh/id_rsa.pub

#on dev env
> su - grader
> mkdir .ssh
> chmod 700 .ssh

> nano .ssh/authorized_keys 
# copy id_rsa contents into authorized_keys, save and quit 

# back to root 
> exit

# disable SSH root access
> nano /etc/ssh/sshd_config 
# set PermitRootLogin to 'no' , save and quit 

# reload SSH
> service ssh restart

# on localhost - check that I can login as a grader user 
> ssh grader@52.26.76.149

# as the grader user
# update packages (5.)
> sudo apt-get update

# change SSH port to 2200 (6.)
> sudo nano /etc/ssh/sshd_config
# change Port 22 to Port 2200 , save & quit
# reload SSH
> sudo service ssh restart
# check that change worked out 
> netstat -an | grep "LISTEN "
tcp        0      0 0.0.0.0:2200            0.0.0.0:*               LISTEN     
tcp6       0      0 :::2200                 :::*                    LISTEN     

# enable UFW
> sudo ufw enable

# make sure that IPv6 is supported by UFW
> sudo nano /etc/default/ufw 
# look for IPV6=yes 

# allow SSH:2200 (7.1)
> sudo ufw allow 2200/tcp
# allow HTTP:80 (7.2)
> sudo ufw allow www
# allow NTP (7.3)
> sudo ufw allow ntp

# check UFW status
> sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123                        ALLOW       Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123 (v6)                   ALLOW       Anywhere (v6)

# check local timezone 
> date +%Z
UTC 
# timezone is already set to UTC - do nothing (8. ?)

# installing apt packages
> sudo apt-get upgrade

# installing apache web server (9.1)
> sudo apt-get install apache2
# installing mod_wsgi (9.2)
> sudo apt-get install python-setuptools libapache2-mod-wsgi
# restarting apache
> sudo service apache2 
#check that Python is installed
> python --version
Python 2.7.6

# installing PostgreSQL
> sudo apt-get install postgresql

# changing PostgreSQL authentication file (http://www.depesz.com/2007/10/04/ident/)
> sudo pico /etc/postgresql/9.3/main/pg_hba.conf 
... 
local   all             all                                     md5
...

# creating catalog user (10.2)
> sudo -u postgres createuser --interactive catalog

# adding password for catalog postgresql user
> sudo su - postgres
> psql
> ALTER ROLE catalog WITH PASSWORD '...'; 
 
 # install GIT
 > sudo apt-get install git
 
 # cloning udacity-p3 repo
 > git clone https://github.com/pzmudzinski/udacity-p3.git
 
 > cd udacity-p3/vagrant 
 > sudo apt-get -qqy update
 > sudo apt-get -qqy install postgresql python-psycopg2
 > sudo apt-get -qqy install python-flask python-sqlalchemy
 > sudo apt-get -qqy install python-pip
 > sudo pip install coverage Flask-Login flask-marshmallow Flask-SQLAlchemy Flask-WTF rauth marshmallow-sqlalchemy bleach oauth2client requests httplib2
 
 # creating catalog db 
 > createdb -U catalog --locale=en_US.utf-8 -E utf-8 -O catalog app -T template0
 
 # changing config.py file for new db's location
 # SQLALCHEMY_DATABASE_URI = 'postgresql://catalog:...@localhost/app'

# creating VirtualHost
> sudo nano /etc/apache2/sites-available/catalog.conf 
<VirtualHost *:80>
                ServerName 52.26.76.149
                ServerAdmin admin@mywebsite.com
                WSGIScriptAlias / /var/www/catalog/app.wsgi
                <Directory /var/www/catalog/app/>
                        WSGIProcessGroup app
                        WSGIApplicationGroup %{GLOBAL}
                        WSGIScriptReloading On
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/catalog/app/static
                <Directory /var/www/catalog/app/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

# creating .wsgi File
> sudo nano /etc/apache2/sites-available/catalog.conf 
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog")

from app import app as application
application.secret_key = '...'

# restart Apache
> sudo service apache2 restart

```

## Resources used 
* http://killtheyak.com/use-postgresql-with-django-flask/
* https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
* http://www.depesz.com/2007/10/04/ident/
* http://stackoverflow.com/questions/20212894/how-do-i-get-flask-to-run-on-port-80
* https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps
* and many many others... 