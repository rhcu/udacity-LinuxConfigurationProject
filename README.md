# udacity-LinuxConfigurationProject
Linux Server Configuration with Lightsail AWS 

## The IP address and SSH port so server can be accessed by the reviewer.
IP address - `35.177.254.104`
SSH port - 2200

## The complete URL to the hosted web application.

DNS address - `http://ec2-35-177-254-104.eu-west-2.compute.amazonaws.com`

## A summary of software installed and configuration changes made.

### Step 1: Start an Amazon Lightsail instance
* Create an AWS account and Log in to Lightsail

* Create an instance, specify that you need OS only Ubuntu image, give it the hostname (it won't be displayed anywhere), click `Create`

* Wait some time for the instance to start. You can find your public and private IP in `Networking` section now


### SSH into your server from your local machine (I have Linux Ubuntu)
* Download the private key in your account section. It will have `.pem` extension

* On your local machine, move this key into newly created directory `~/.ssh` with `udacity_key.rsa` name. 

* Change permission of the file with `chmod 600 ~/.ssh/udacity_key.rsa` to prevent other users to be able to edit of read it.

* You are able to connect to your server with the following command now: `ssh -i ~/.ssh/udacity_key.rsa ubuntu@35.177.254.104`, where 35.177.254.104 is my Public IP.

* You will see a prompt asking about reliability of this server, answer `yes`.

### Security: update packages, change port, configure UFW, install Fail2Ban and automatic packages update
* Run the following commands to update and upgrade packages
⋅⋅⋅ `sudo apt-get update`
⋅⋅⋅ `sudo apt-get upgrade`

* Edit the `sshd_config` file: `sudo vi /etc/ssh/sshd_config` to change Port 22 to Port 2200

* Restart ssh with `sudo service ssh restart` command

* Configure UFW with the following commands:
⋅⋅⋅ ```sudo ufw default deny incoming

sudo ufw default allow outgoing

sudo ufw allow 2200/tcp

sudo ufw allow 80/tcp 

sudo ufw allow 123/udp

sudo ufw enable```
iv. A list of any third-party resources you made use of to complete this project
1 - Update the server packages: 
sudo apt update
sudo apt upgrade
sudo apt autoremove
2 - Change SSH port from 22 to 2200
sudo nano /etc/ssh/sshd_config # change here and save file 
sudo service ssh restart  # for changes to take power
3 - Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp 
sudo ufw allow 123/udp
sudo ufw enable
4 - Create a new user, give permission to sudo and create key-pair
sudo adduser grader # 'udacity' password; other values are default
sudo usermod -aG sudo grader # gives grader permission to sudo
ssh-keygen # public key saved with empty password in default folder /home/ubuntu/.ssh/id_rsa.pub
5 - Configure the local timezone to UTC
sudo dpkg-reconfigure tzdata
6 - Install and configure Apache to serve a Python mod_wsgi application
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi-py3
7 -  Install and configure PostgreSQL
sudo apt install postgresql 
check that there are no remote connections in this file
sudo nano /etc/postgresql/9.5/main/pg_hba.conf
8 - Install git and clone Item Catalog project
sudo apt install git


