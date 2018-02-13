# Linux Configuration Project
Linux Server Configuration with Lightsail AWS 

## The IP address and SSH port so server can be accessed by the reviewer.
IP address - `35.177.254.104`
SSH port - 2200

## The complete URL to the hosted web application.

DNS address - http://ec2-35-177-254-104.eu-west-2.compute.amazonaws.com

## A summary of software installed and configuration changes made.

### Step 1: Start an Amazon Lightsail instance
* Create an AWS account and Log in to Lightsail

* Create an instance, specify that you need OS only Ubuntu image, give it the hostname (it won't be displayed anywhere), click `Create`

* Wait some time for the instance to start. You can find your public and private IP in `Networking` section now


### Step 2: SSH into your server from your local machine (I have Linux Ubuntu)
* Download the private key in your account section. It will have `.pem` extension

* On your local machine, move this key into newly created directory `~/.ssh` with `udacity_key.rsa` name. 

* Change permission of the file with `chmod 600 ~/.ssh/udacity_key.rsa` to prevent other users to be able to edit of read it.

* You are able to connect to your server with the following command now: `ssh -i ~/.ssh/udacity_key.rsa ubuntu@35.177.254.104`, where 35.177.254.104 is my Public IP.

* You will see a prompt asking about reliability of this server, answer `yes`.

### Step 3: Security: update packages, change port, configure UFW, install Fail2Ban and automatic packages update
* Run the following commands to update and upgrade packages
⋅⋅⋅ `sudo apt-get update`
⋅⋅⋅ `sudo apt-get upgrade`
⋅⋅⋅ `sudo apt autoremove`

* Edit the `sshd_config` file: `sudo vi /etc/ssh/sshd_config` to change Port 22 to Port 2200. This is done to somehow prevent attacks on the default port.

* Restart ssh with `sudo service ssh restart` command

* Configure UFW with the following commands to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).:
``` 
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp 
sudo ufw allow 123/udp
sudo ufw enable
```
You can check the status of your firewall with `sudo ufw enable`

* Exit SSH connection with `exit` and configure ports in AWS Lightsail to add Custom TCP port 2200 and Custom UDP Port 123, and to delete SSH 22 port. After that try to connect with the `ssh -i ~/.ssh/udacity_key.rsa ubuntu@35.177.254.104`, where 
35.177.254.104 is my Public IP, command from the previous step. If you are unable to do this, try to connect using another 
Internet connection, or you have problems and locked off your server. :(

* SSH creates a secure server; however, it is exposed to attacks. Hence, we can use Fail2Ban, which automatically changes
firewall settings to prevent attacks. It sends an email when someone is banned. Follow the commands below to install and 
configure it.
```
sudo apt-get install fail2ban
sudo apt-get install sendmail iptables-persistent
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```
Configure settings in local file according to your needs. For example, if you want the email to include the relevant log 
lines, you can change action item in the jail.local file to `action_mwl` and to add your destination email.
Change under `[sshd]` `port = ssh` to `port = 2200`, as we have changed the default one.

* One more way to increase security is to configure automatic updates 
```
sudo apt-get install unattended-upgrades 
sudo vi /etc/apt/apt.conf.d/50unattended-upgrades # open this file and uncomment ${distro_id}:${distro_codename}-updates
sudo vi /etc/apt/apt.conf.d/20auto-upgrades # change file to install updates after the needed time interval, e.g. daily.
sudo dpkg-reconfigure --priority=low unattended-upgrades # enable it
# some packages can be not updated asking to restart system, so use this:
sudo apt-get update 
sudo apt-get dist-upgrade
sudo shutdown -r now
```
### Step 4: Add grader user
* generate key `ssh_key` on local machine using `ssh-keygen` and save them in `~/.ssh`, where previously `udacity_key.rsa` was saved
* open this file and copy the content
* create a file on your virtual machine and paste the content there 
```
su - grader
mkdir .ssh
vim .ssh/authorized_keys
```
* give grader `sudo` access using `sudo visudo` adding `grader  ALL=(ALL:ALL) ALL` line to the opened file.
* log in as grader with `ssh -i ~/.ssh/ssh_key -p 2200 grader@35.177.254.104`

### Step 5: Configure time
* Use this command to set time: `sudo dpkg-reconfigure tzdata`

### Step 6: Install Apache
Following commands are for Python 2.7
* `sudo apt-get install apache2` # installs Apache
* `sudo apt-get install python-setuptools libapache2-mod-wsgi`# installs mod_wsgi
* `sudo service apache2 restart`

### Step 7: Install and configure PostgreSQL
* Run the following commands 
```
sudo apt-get install postgresql
sudo vim /etc/postgresql/9.5/main/pg_hba.conf # Check if no remote connections are allowed
sudo su - postgres # Login as "postgres"
psql 
```
* Create a new database named catalog and a new user catalog with catalog password in postgreSQL shell 
```
postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
postgres=# ALTER ROLE catalog CREATEDB;
\q
exit
``` 
* Create a new Linux user catalog and give sudo permissions (same instructions as for grader user)
* Log in as `catalog` and create `catalog` database `createdb catalog`; `exit` to return to `grader`
### Step 8: Install git 
`sudo apt-get install git`
### Step 9: Clone project from GitHub
* Create `/var/www/catalog/` directory
* Change directory to the above
* Clone the project `sudo git clone https://github.com/rhcu/Item-Catalog-Udacity-Project4.git catalog`
* Change ownership to grader `sudo chown -R grader:grader catalog/`
* Rename `application.py` to `__init__.py` 
* Change in `__init__.py` the last line to `app.run()` 
* Change in all `.py` files `create_engine()` to `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
### Step 10: Change Google Credentials for login
* Create new Credentials with your IP and DNS as JavaScript origins and add `YOUR_DNS_HERE/oauth2callback` to redirect URI
* Change `client_secrets.json` in your app directory to new one and `client_id` in `login.html`
* In `__init__.py` there could be a problem while reading `.json` file, so change the line that loads info from it with the following `result = json.loads(h.request(url, "GET")[1].decode("utf-8"))`

### Step 11: Configure and Enable a New Virtual Host
* `sudo apt-get install python-virtualenv` # install virtual environment
* In the directory of your app, create a new env `sudo virtualenv -p python venv3`
* Change the ownership of this environment to `grader` 
* Activate it `. venv3/bin/activate`
* Install needed dependencies that you have used in your project
```
pip install httplib2
pip install requests
pip install --upgrade oauth2client
pip install sqlalchemy
pip install flask
sudo apt-get install libpq-dev
pip install psycopg2
```
There could be some slight changes in the syntaxis of these commands, especially in psycopg2 as it currently stopped maintenance for Python 2.7. Install it according to suggestion of Shell (e.g. binary file)
* Run `__init__.py` to see if there are no mistakes at that point. 
* `deactivate` environment

### Step 12: Install Flask app configurations
* `sudo vi /etc/apache2/sites-available/FlaskApp.conf` # open this file to edit
* Add the following lines of code
```
<VirtualHost *:80>
		ServerName 35.177.254.104
    ServerAlias http://ec2-35-177-254-104.eu-west-2.compute.amazonaws.com
		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
		<Directory /var/www/catalog/catalog/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
```
* Enable the virtual host `sudo a2ensite catalog` and `sudo service apache2 reload` to reload Apache
* Create `/var/www/catalog/catalog.wsgi` file and add the following text there
```
activate_this = '/var/www/catalog/catalog/venv3/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalog/")
sys.path.insert(1, "/var/www/catalog/")

from catalog import app as application
application.secret_key = "some_very_difficult_key_to_protect_data"
```
* `sudo service apache2 restart` 
### Step 14: Database setup
* add following lines to the `.py` file that populates DB
```
import sys
sys.path.insert(0, "/var/www/catalog/catalog/venv3/lib/python2.7/site-packages") 
```
* Activate `venv3` and run the code to populate db, deactivate the environment 

### Step 15: Displaying the app
* Deactivate default Apache site `sudo a2dissite 000-default.conf`
* Reload Apache
* Change ownership to be able to display: `sudo chown -R www-data:www-data catalog/`
* Restart Apache Server
* Visit your DNS. Here http://ec2-35-177-254-104.eu-west-2.compute.amazonaws.com

### Resources 
* DigitalOcean Fail2Ban Configuration for Ubuntu: https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04
* Very useful and well-formated README: https://github.com/boisalai/udacity-linux-server-configuration
* One more useful GitHub: https://github.com/kongling893/Linux-Server-Configuration-UDACITY/blob/master/README.md
* PostgreSQL Documentation: https://www.postgresql.org/docs/9.0/static/sql-createdatabase.html
* Student Book relevant to this project (already deprecated but has links still): https://udacity.atlassian.net/wiki/spaces/BENDH/pages/7930043/Project+7+Linux+Server+Configuration
* Information about SSH to get an understanding: https://www.ssh.com/ssh/protocol/
* Udacity SSH Webcast: https://www.youtube.com/watch?v=HcwK8IWc-a8
