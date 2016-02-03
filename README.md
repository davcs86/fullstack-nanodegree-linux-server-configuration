# Project #5 by davcs86

URL: [http://ec2-54-69-180-83.us-west-2.compute.amazonaws.com/index](http://ec2-54-69-180-83.us-west-2.compute.amazonaws.com/index)

SSH: -p 2200 grader@54.69.180.83

# Criteria # 1: User management

### - Create a new user named grader and grant this user sudo permissions.

```bash
$> adduser grader
$> usermod -a -G sudo grader
```

### - Give ssh acccess to grader

Create a local key `grader_udacity_key.rsa`

```bash
$> ssh-keygen -t rsa
```

Upload the `grader_udacity_key.rsa.pub` to the server with the root key

```bash
$> cat ~/.ssh/grader_udacity_key.rsa.pub | ssh -i ~/.ssh/udacity_key.rsa root@54.69.180.83 "mkdir /home/grader/.ssh && cat >> /home/grader/.ssh/authorized_keys"
```

**Logout from root account, then use the grader account**

```bash
$> exit
$> ssh -i ~/.ssh/grader_udacity_key.rsa grader@54.69.180.83
```

# Criteria # 2: Security

### - Disable remote login to root

With grader account

```bash
$> sudo nano /etc/ssh/sshd_config
   edit the options
   Port to 2200
   and 
   PermitRootLogin to no
```
  
restart ssh service

```bash
$> sudo service ssh restart
```

### - Install updates

```bash
$> sudo apt-get update
$> sudo apt-get upgrade
   For automatic package updates, install cron-apt
$> sudo apt-get install cron-apt
```

### - UTC timezone (and synchronization with NTP)

```bash
$> sudo tzselect
   Select option 12) TZ - I want to specify the time zone using the Posix TZ format.
   Write 'UTC-0', enter
   Confirm the change with 1, enter
```
NTP synchronization
```bash
$> sudo apt-get install ntp
```  

### - UFW Firewall & Fail2Ban to block multiple unsuccessful login attempts.

UFW

```bash
$> sudo apt-get install ufw
   deny all incoming connections by default
$> sudo ufw default deny incoming
   allow all outgoing connections by default (optional)
$> sudo ufw default allow outgoing
   allow ssh, http and ntp services
$> sudo ufw allow 2200/tcp
$> sudo ufw allow 80/tcp
$> sudo ufw allow 123/tcp
   enable the ufw firewall
$> sudo ufw enable
```

Fail2Ban

```bash
$> sudo apt-get install fail2ban
   copy the default settings
$> sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
$> sudo nano /etc/fail2ban/jail.local
   edit the options
   backend to polling
   and in the [ssh] section
   port to 2200
$> sudo service fail2ban restart
$> sudo iptables -L
   restart
$> sudo reboot
   check the banned ips with
$> sudo fail2ban-client status ssh
```

## Criteria # 3: Application functionality

### - Apache2 and WSGI support

```bash
$> sudo apt-get install apache2 libapache2-mod-wsgi python-dev
   enable wsgi support
$> sudo a2enmod wsgi   
```

### - Install Postgresql and create user and database

```bash
$> sudo apt-get install postgresql
   switch to postgresql user
$> sudo su - postgres
$> createuser --interactive
   Enter name of role to add: catalog
   Shall the new role be a superuser? (y/n) n
   Shall the new role be allowed to create databases? (y/n) n
   Shall the new role be allowed to create more new roles? (y/n) n
   CREATE ROLE
$> createdb catalog
   CREATE DATABASE
$> psql
postgres=# alter user catalog with encrypted password 'catalog123';
   ALTER ROLE
postgres=# grant all privileges on database catalog to catalog;
   GRANT
postgres=# \q
$>
```

**Switch back to grader account**

```bash
$> exit
```

Check that no remote connections are allowed by looking in the host based authentication file:

```bash
$> sudo nano /etc/postgresql/9.1/main/pg_hba.conf
   should display something like this (must have no remote wildcard like 0.0.0.0, or specific IP like 122.122.122.122)
   local   all             postgres                                peer
   local   all             all                                     peer
   host    all             all             127.0.0.1/32            md5
   host    all             all             ::1/128                 md5
```

### - Download (and configure) the Item catalog app 

```bash
   before, install git client
$> sudo apt-get install git
   clone the git repository in /var/www/
$> sudo git clone https://github.com/davcs86/fullstack-nanodegree-flask-item-catalog.git
   install the app dependencies, watch /var/www/fullstack-nanodegree-flask-item-catalog/vagrant/pg_config.sh
$> sudo apt-get install python-psycopg2 python-flask python-sqlalchemy python-pip python-dev libmagickwand-dev npm git nodejs-legacy
   install the pip dependencies
$> sudo pip install oauth2client requests httplib2 passlib flask-httpauth flask-login rauth flask-seasurf flask-sqlalchemy wtforms awesome-slugify SQLAlchemy-ImageAttach Werkzeug
   change the file permissions
$> sudo chmod 777 -R .
   install Bower globally, to install the frontend dependencies of the project
$> sudo npm install -g bower
   to install the frontend dependencies of the project
$> cd fullstack-nanodegree-flask-item-catalog/vagrant/catalog/app/static
$> bower install
   configure the database connection in 
$> cd /var/www
$> nano fullstack-nanodegree-flask-item-catalog/vagrant/catalog/app/config.py
   line SQLALCHEMY_DATABASE_URI = "postgresql://catalog:catalog123@localhost/catalog"
   configure the app to use the production settings
$> nano fullstack-nanodegree-flask-item-catalog/vagrant/catalog/app/app_setup.py
   from .config import DevelopmentConfig as app_config  to  from .config import ProductionConfig as app_config
   and
   fs_store = HttpExposedFileSystemStore('app/static', 'images/') to fs_store = HttpExposedFileSystemStore('/var/www/fullstack-nanodegree-flask-item-catalog/vagrant/catalog/app/static', 'images/')
```

### - Create the .wsgi app

```bash
$> nano fullstack-nanodegree-flask-item-catalog/vagrant/catalog/catalogApp.wsgi
   write
   #!/usr/bin/python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0,"/var/www/fullstack-nanodegree-flask-item-catalog/vagrant/catalog/")
   from app import app as application
   if __name__ == '__main__':
      application.run(host=application.config['HOST'],
              port=application.config['PORT'],
              debug=application.config['DEBUG'])
```

### - Configure and Enable a New Virtual Host

```bash
   Create the virtual host
$> sudo nano /etc/apache2/sites-available/CatalogApp.conf
   write the settings
   <VirtualHost *:80>
		ServerName ec2-XX-XX-XXX-XXX.us-west-2.compute.amazonaws.com
		ServerAdmin davcs86@gmail.com
		WSGIScriptAlias / /var/www/fullstack-nanodegree-flask-item-catalog/vagrant/catalog/catalogApp.wsgi
		<Directory /var/www/fullstack-nanodegree-flask-item-catalog/vagrant/catalog/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/fullstack-nanodegree-flask-item-catalog/vagrant/catalog/app/static
		<Directory /var/www/fullstack-nanodegree-flask-item-catalog/vagrant/catalog/app/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   Disable the default site, and enable the catalog app
$> sudo a2dissite 000-default
$> sudo a2endsite CatalogApp
   Restart apache2 service
$> sudo service apache2 restart
```
## Install NewRelic as monitoring tool

```bash
   Install application monitoring agent and generate the settings
$> sudo pip install newrelic
$> newrelic-admin generate-config de403226fbf52697b26124f59923897XXXXXXXXX /var/www/fullstack-nanodegree-flask-item-catalog/vagrant/catalog/newrelic.ini
   Modify your .wsgi file
$> nano /var/www/fullstack-nanodegree-flask-item-catalog/vagrant/catalog/catalogApp.wsgi
   write above of "from app import app as application"
   import newrelic.agent
   newrelic.agent.initialize('/var/www/fullstack-nanodegree-flask-item-catalog/vagrant/catalog/newrelic.ini')
   then, restart apache2 service
$> sudo service apache2 restart
   Install server monitoring agent
$> sudo nano /etc/apt/sources.list.d/newrelic.list
   write the new relic repository
   deb http://apt.newrelic.com/debian/ newrelic non-free
$> wget -O- https://download.newrelic.com/548C16BF.gpg | sudo apt-key add -
$> sudo apt-get update
   install the agent package
$> sudo apt-get install newrelic-sysmond
   create the settings
$> sudo nrsysmond-config --set license_key=de403226fbf52697b26124f59923897XXXXXXXXX
   start the agent service
$> sudo service newrelic-sysmond start
$> sudo reboot
```
