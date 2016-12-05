# Udacity-Linux-Server-Configuration


IP address: 35.163.43.2
SSH port: `2200`
URL: http://ec2-35-163-43-2.us-west-2.compute.amazonaws.com/

## Set up remote machine

- Add a new user called grader: `adduser grader`
- Give the new user sudo privilege: `usermod -aG sudo grader`
- Update package index: `apt-get update`
- Upgrade the installed packages: `apt-get upgrade`

## Setup SSH keys for user grader
- In the local machine: `ssh-keygen -t rsa`
- SCP public key to the remote machine: `scp -p 22 ~/.ssh/id_rsa.pub grader@<ip-address>:~/.ssh`
- On server machine
	```
	cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
	chmod 700 .ssh
	chmod 600 .ssh/authorized_keys
	rm .ssh/id_rsa.pub
	```

## Fix sudo resolve host error
`echo $(hostname -I | cut -d\  -f1) $(hostname) | sudo tee -a /etc/hosts`

## Disable root login and other security settings
- Open ssh config file: `sudo vim /etc/ssh/sshd_config`
- Chage ssh port to 2200
- Change PasswordAuthentication to no, changed PermitRootLogin to no and PrintLastLog to no
- Changed LoginGraceTime to 60, MaxStartups to 2, X11Forwarding to no and LogLevel to VERBOSE

## Change timezone to UTC
- Open the timezone selection dialog:  `sudo dpkg-reconfigure tzdata`
- Then chose 'None of the above', then UTC.
-  Setup the ntpd for regular time sync:  `sudo apt-get install ntp`
- Choose closer NTP time servers and put in /etc/ntp.conf

## Install and configure Apache 
- apache web server: `sudo apt-get install apache2`
- Install mod_wsgi for serving Python apps from Apache: `sudo apt-get install python-setuptools libapache2-mod-wsgi`
- Restart apache: `sudo service apache2 restart`  

## Install git 
- Install git: `sudo apt-get install git`
-  Set up name and email for git : `git config --global user.name <name>` and `git config --global user.email <email>`

## Set up remote machine for flask application
- Install additional packages to enable Apache to serve Flask applications:  `sudo apt-get install libapache2-mod-wsgi python-dev`
- Create directory for the flask app: `cd /var/www` 
- Create directory catalog 
- Clone the movie catalog from git inside this directory and renamed the top subdirectory as catalog
- Install python-pip: `sudo apt-get install python-pip` 
- Install virtualenv: `sudo pip install virtualenv`
- Set virtual environment to name 'venv':  `sudo virtualenv venv`
-  Create a virtual host config file : `sudo nano /etc/apache2/sites-available/catalog.conf`
- Edit the file as per project settings 
- Enabled the host : `sudo a2ensite catalog`
- Create the .wsgi file `catalog.wsgi` as per project settings.
- Restarted apache: `sudo service apache2 restart`

## Made the git repo web inaccessible
- Create and open .htaccess file: `cd /var/www/catalog/` and `sudo vim .htaccess`
- Pasted this line of code inside: `RedirectMatch 404 /\.git`

## Set up the virtual environment for testing project
- Activate virtual environment: `source venv/bin/activate`
- Install required packages like httplib2, requests, flask-seasurf, oauth2.client, sqlalchemy and python-psycopg2 inside the virtual 
environment

##Configure PostgreSQL
- Install PostgreSQL:  `sudo apt-get install postgresql postgresql-contrib`
- Edit database_setup.py to and edit the create_engine line:   
  ```python engine =create_engine('postgresql://catalog:<pwd>@localhost/catalog')```
- Change the same line in project.py file
- Create catalog user for postgresql: `sudo adduser catalog`
- Change to default user postgres:`sudo su - postgres`
- Connect to the system and create user with LOGIN role and set a password: `CREATE USER catalog WITH PASSWORD '<pwd>';
- Allow the user to create database tables:  `ALTER USER catalog CREATEDB;`
- Create database:  `CREATE DATABASE catalog WITH OWNER catalog;`
- Revoke all rights and grant only access to the catalog role
- Create database: `python database_setup.py`

## Running application and debugging 
- Restart Apache:  `sudo service apache2 restart`
- In case of error we can see last few lines of error log as in:`sudo tail -10 /var/log/apache2/error.log`

## OAuth-Login
- Login to Google developer console and navigate to the project
- Add public url to the Javascript origins and hostname

## Glances
- Install glances: `sudo apt-get install glances`
- Edit glances to monitor apache and postgres

## Fail2Ban
- Install fail2Ban: `sudo apt-get install fail2ban`
- Install sendmail: `sudo apt-get install sendmail`
- Copy the Fail2Ban configuration file to a local config file that will override it: `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
- Make changes in the default and ssh sections 
- Define a new action for fail2Ban as `/etc/fail2ban/action.d/ufw-ssh.conf` 
- Stop and start fail2Ban service for the changes to take effect.


### Helpful Links
1. http://www.tutorialspoint.com/unix_commands/aureport.htm
2. https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
3. https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart
4. https://discussions.udacity.com/t/apache2-flask-postgresql-error/36724/3
5. https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04
6. https://www.digitalocean.com/community/tutorials/how-to-deploy-python-wsgi-applications-using-uwsgi-web-server-with-nginx

