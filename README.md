
# Project: Linux Server Configuration

Linux Server Configuration is a project in the Full Stack Web Developer Nanodegree Program, This project helps in building a Linux server from scratch. The Linux server is built  upon the Amazon's lightsail server instance. 

# Server Overview
 The server is an very basic  instance with 512 MB RAM, 1 vCPU, 20 GB SSD. This is sufficient for my Item Catalog project. The Linux flavor I have  used is: 

 - Distributor ID: Ubuntu 
 - Description:    Ubuntu 16.04.6 LTS 
 - Release:   16.04 
 - Codename:       xenial

### Server Info
1. Start a new Ubuntu Linux server instance on [Amazon Lightsail](https://lightsail.aws.amazon.com). There are full details on setting up your Lightsail instance on the next page

Private IP: 172.26.12.254
Public IP: **13.127.73.63**  
SSH port: 2200  
Site URL: [http://13.127.73.63/catalog/](http://13.127.73.63/catalog/)
Server Status: **Running**
Git: https://github.com/vinayrajan/serverconfig

# Linux Configuration
2. Follow the instructions provided to SSH into your server.
Downloaded the default private key to the local machine and logged in successfully using putty

3. Update all currently installed packages.
Upgraded the system to get the latest security patches.
`sudo apt-get update`
`sudo apt-get upgrade`

6. Create a new user account named `grader`.  
7. Give `grader` the permission to `sudo`.  
8. Create an SSH key pair for `grader` using the `ssh-keygen` tool.
Create User named 
Created a user called grader and added to **sudo users**
```sudo adduser grader```
```sudo touch /etc/sudoers.d/grader```

Created a ssh key pair using putty's ssh-keygen tool and copied the private key to ~/.ssh folder
```
su - grader
mkdir .ssh
touch .ssh/authorized_keys
nano .ssh/authorized_keys
```
change the access of the .ssh folder and contents within the folder
```
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```
4. Change the SSH port from **22** to **2200**. Make sure to configure the Lightsail firewall to allow it.
 Changed the SSh port from ### 22 to 2200
 `sudo nano /etc/ssh/sshd_config`
 using the sshd_config file i changed the port 22 to 2200
 `sudo service ssh restart`
 
5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

```
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow 123/udp
sudo ufw deny 22
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
sudo ufw status
```
using the above commands i could easily configure the Uncomplicated Firewall (UFW) allowing the incomming connections or SSH (port 2200), HTTP (port 80), and NTP (port 123) only.

## Configuring Web Server
Installing Apache
`sudo apt-get install apache2`
Testing Apache using the IP address shows the default Apache homepage.
http://13.127.73.63/ or http://ec2-13-127-73-63.ap-south-1.compute.amazonaws.com

#### Install python3 and  configure mod_wsgi package
The server had inbuilt python2.7 and had to install python3 and pip3
`sudo apt-get install python3`
`sudo apt-get install python3-pip`
`sudo pip3 -r requirements.txt` 

Python was running and python2 catalog.py worked without errors

Then installed
`sudo apt-get install libapache2-mod-wsgi`

## Configuring Database
Installed PostgresSql and configured using the following commands
`sudo apt-get install postgresql`
Create a new database and user named `catalog` with password  and set permissions to the database database.

## Configuring Website
Created a config on apache2 on `/etc/apache2/sites-enabled/catalog.conf` for the item catalog project
```
<VirtualHost *:80>
     # Add machine's IP address (use ifconfig command)
     #ServerName 13.127.73.63
     ServerName ec2-13-127-73-63.ap-south-1.compute.amazonaws.com
     # Give an alias to to start your website url with
     WSGIScriptAlias /catalog /home/ubuntu/ExampleFlask/my_flask_app.wsgi
     <Directory /home/ubuntu/ExampleFlask/>
        # set permissions as per apache2.conf file
            Options FollowSymLinks
            AllowOverride None
            Require all granted
     </Directory>
     ErrorLog ${APACHE_LOG_DIR}/example_error.log
     LogLevel warn
     CustomLog ${APACHE_LOG_DIR}/example_access.log combined
</VirtualHost>
```

## Testing
Installed Lynx browser to test the site internally.
Changed the **OAuth 2.0 client ID** from  console.developers.google.com


# Website Urls

### Home Page
http://ec2-13-127-73-63.ap-south-1.compute.amazonaws.com/catalog/

### Json Endpoints
http://ec2-13-127-73-63.ap-south-1.compute.amazonaws.com/catalog/products/json
http://ec2-13-127-73-63.ap-south-1.compute.amazonaws.com/catalog/product/1/json
http://ec2-13-127-73-63.ap-south-1.compute.amazonaws.com/catalog/categories/json
http://ec2-13-127-73-63.ap-south-1.compute.amazonaws.com/catalog/category/1/json
    
Thankyou
Vinay Kumar Rajan
vr3924@intl.att.com
