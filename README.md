# Linux Server Configuration - Coffeeshop website depolyment
IP adderess and SSH ports:
- IP address: 54.85.177.225
- SSH port : 2200
- URL for my website: 54.85.177.225.xip.io
# The complete process and configurations done:
### Ubuntu setup:
- Created an instance in Ubuntu.
- Downloaed the `.pem` extension folder by going to Account settings.
- Used puTTy, and puTTygen to ssh locally to my server and the can be done by following the guide in the following link:
  - https://lightsail.aws.amazon.com/ls/docs/en/articles/lightsail-how-to-ssh-connect-to-instance-virtual-private-server-using-putty
- now I can access my instance using my local ssh terminal.

### Port configuration and setup:
- run `sudo apt-get update` and `sudo apt-get upgrade` to update all currently installed packages.
- run the follwoing commands to only allow incoming connections to port 2200 (SSH), 80 (HTTP) and 123 (NTP):
  - `sudo ufw defualt deny incoming`
  - `sudo ufw defualt allow outgoing`
  - `sudo ufw allow 2200/tcp`
  - `sudo ufw allow http`
  - `sudo ufw allow ntp`
  - `sudo ufw enable`
- Go to my lightsail account and update the ports accordingly.

### Configuring the local timezone to UTC:
(help provided by the Slack community)
- run `sudo dpkg-reconfigure tzdata` 
- choose None of the above when prompted and then UTC.

### Downloading all the libraries needed:
- run `sudo apt-get install apache2` - For apache server depolyment.
- run `sudo apt-get install libapache2-mod-wsgi` - .wsgi file needed (coming)
- run `sudo apt-get install postgresql postgresql-contrib` - for postgresql (Database)
- run `sudo apt-get install python-psycopg2 python-flask` - for psycopg2, and flask libraries used in my project
- run `sudo apt-get install python-sqlalchemy python-pip` - for sqlalchemy
- export LC_ALL=C - forcing the application to use the defualt language 
- run `sudo apt-get install git` - for git repository

#### Here through many questions I asked on slack and to friends I realized the need to start a virtual machine because I noticed a consistant error of not having flask installed, so I did the follwing:
(help from Ebtihal in slack community who provided me this link: https://github.com/chuanqin3/udacity-linux-configuration)
- run `sudo pip install virtualenv` - install virtual machine
- run `sudo virtualenv venv` - rename the virtual machine to venv
- run `source venv/bin/activate` - activate the virtual machine
- run `sudo chmod -R 777 venv` - give file access permissions
- run inside the venv:
  - `sudo apt-get install python-pip`
  - `sudo pip install Flask`
  - `sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils requests render_template redirect psslib`

### Adding the grader user:
- run `sudo adduser grader` and filled the password: `udacity`
- run `sudo usermod -aG sudo grader` - giving grader permission to sudo
- Now I moved to my local terminal to generate the key for grader to enable SSH:
  - run `ssh-keygen`
  - saved it and named it "udacityKey"
  - open udacityKey.pub
  - copy the SSH rsa key.
- go back to the server terminal
- run `sudo mkdir .ssh`
- run `sudo touch .ssh/authorized_keys`
- run `sudo nano .ssh/authorized_keys`
- now I paste the public key I got from `udacityKey.pub`
- change the file permissions and owner by running:
  - `sudo chmod 700 /home/grader/.ssh`
  - `sudo chmod 644 /home/grader/.ssh/authorized_keys`
  - `sudo chown -R grader:udacity /home/grader/.ssh`
- restart the SSH service `sudo service ssh restart`
- now we can log in with grader using the following command on our local terminal:
  - `ssh grader@54.85.177.225 -p 2200 -i ~/.ssh/udacityKey`

### Disable the root login:
- run `sudo nano /etc/ssh/sshd_config`
- change: `PermitRootLogin without-password` to `PermitRootLogin no`

### Setting up the database:
- run `sudo -u postgres createuser -P catalog` - catalog user created
- run `sudo -u postgres createdb -O catalog` - catalog database created and the password is 'udacity'

### cloning item catalog project (Coffeeshop website):
- run `cd /var/www` - from udacity videos
- run `sudo mkdir catalog` to make the directory catalog that we need to clone our project in.
- cd into `catalog`
- run `sudo git clone https://github.com/HaithamAlsharif/fullstack-nanodegree-vm` this will clone the whole vagrant virtual machine, so we need to extract the catalog folder and move it to the catalog directory we just created and that can be done by the following:
  - cd into fullstack-nanodegree-vm/vagrant/catalog
  - run `sudo mv catalog /var/www/catalog`
### Make .git file inaccessable
(from the slack community @Muneera)
- run `sudo nano .htaccess`
- add `RedirectMatch 404 /\.git`

### Prepare for deployment:
- run `sudo mv project.py __init__.py` - changing the name of the project.py to the default and automatically detected `__init__.py` 
- run `sudo nano __init__.py/database_setup.py/data.py` and change the engine to `engine = create_engine('postgresql://catalog:udacity@localhost/catalog')` and change in /__init__.py/ `client_secrets.json` to `/var/www/catalog/catalog/client_secrets.json`

### COnfigure the website for Google authentication:
- add `xip.io` to the domain in credential OAutho consent screen.
- Add `http://54.85.177.225.xip.io` to authorized Javascript origins in the Google Developers Console settings
- Add `/login`,`/gconnect` and `/callback`
- update `client_secrets.json`

### Create the WSGI file
- run `sudo nano /var/www/catalog/catalog.wsgi`
- the file would include the following:
```##!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application

application.secret_key = '192323261443-rhiopgdtu8a27fb5o0cjs3k6v7da4nmu.apps.googleusercontent.com
```
### Configure apache2
- run `sudo nano /etc/apache2/sites-available/catalog.conf`
- it has the following:
```<VirtualHost *:80>
    ServerName http://54.85.177.225
    ServerAlias catalog ec2-54-85-177-225.us-west-2.compute.amazonaws.com/
    ServerAdmin admin@54.85.177.225
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
- run `sudo a2dissite 000-default.conf` - disable apache defualt virtual host configuration 
- run `sudo a2ensite catalog.conf` -  enable virtual host configuration files
- run `sudo apt-get update && sudo apt-get dist-upgrade` - to update all system packages to most recet version
- run `sudo service apache2 restart`and `sudo service apache2 reload`- to restart and reload apache2
