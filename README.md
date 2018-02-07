# udacity-LinuxConfigurationProject
Linux Server Configuration with Lightsail AWS 
i. The IP address and SSH port so your server can be accessed by the reviewer.
IP address - 18.217.108.149

ii. The complete URL to your hosted web application.
iii. A summary of software you installed and configuration changes made.
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


