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
- 
- !!!!!!!
- export LC_ALL=C
- sudo pip install oauth2client
- sudo pip install requests
- sudo pip install httplib2 
- !!!!!!!!

## Adding grader user:

