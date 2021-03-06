
# Project: Linux Server Configuration

Linux Server Configuration is a project in the Full Stack Web Developer Nanodegree Program, This project helps in building a Linux server from scratch. The Linux server is built  upon the Amazon's lightsail server instance. 

## Server Overview
 The server is an very basic  instance with 512 MB RAM, 1 vCPU, 20 GB SSD. This is sufficient for my Item Catalog project. The Linux flavor I have  used is: 

 - Distributor ID: Ubuntu 
 - Description:    Ubuntu 18.04 LTS 
 - Release:   18.04 


### Server Info
Start a new Ubuntu Linux server instance on [Amazon Lightsail](https://lightsail.aws.amazon.com). 

Private IP: 1**.**.*.**1
Public IP: **52.15.135.223**  
SSH port: 2200 
Domain name: [ec2-52-15-135-223.us-east-2.compute.amazonaws.com](ec2-52-15-135-223.us-east-2.compute.amazonaws.com)
Site URL: [http://ec2-52-15-135-223.us-east-2.compute.amazonaws.com/catalog/](http://ec2-52-15-135-223.us-east-2.compute.amazonaws.com/catalog/)
Server Status: **Running**
Git: https://github.com/vinayrajan/serverconfig

## Linux Configuration

`sudo apt-get update` to update packages
`sudo apt-get upgrade` to install newest versions of packages

> Tip: Many of these commands requires sudo access. Sudo stands for Super User, to become root user type `sudo su - ubuntu` so that you don't need to use sudo in for rest of the commands.
> example instead of `sudo service apache2 restart` you will `service apache2 restart`. 


#### Change the SSH port from 22 to 2200
`sudo nano /etc/ssh/sshd_config`
need to change the line that says
**#Port 22** to **Port 2200**
and save the file.
`sudo service ssh restart` to update the changes 
While you are in the sshd_config file change the line that says
> #PermitRootLogin prohibit-password
to
> #PermitRootLogin no
so that root login is prohibited


#### New user called grader
1. Create a new user account grader:`sudo adduser grader`
2. added grader after root in `sudo nano /etc/sudoers`
3. using `ssh-keygen` created a public key and private key for the user grader in the folder ~/grader/.ssh
4. changed permissions for the folder `sudo chmod 700 .ssh`
5. changed permissions for the files under .ssh folder `chmod 644 .ssh/*`
6. Restart SSH: `sudo service ssh restart`

#### Configure the local timezone to UTC
To change the time zone and set the time zone as utc we need to follow these steps:
`sudo dpkg-reconfigure tzdata`
> here select  None of the above to set timezone to UTC 
```Current default time zone: 'Etc/UTC'
Local time is now:      Fri Jan  3 15:52:52 UTC 2020.
Universal Time is now:  Fri Jan  3 15:52:52 UTC 2020.
```
### Configuring the UFW Firewall
Please note when you create a new instance the firewall will be disabled.
to check the status use `sudo service ufw status`

1. Set default firewall to deny all incoming: 
`sudo ufw default deny incoming`
2. Set default firewall to allow all outgoing: 
`sudo ufw default allow outgoing`
3. Allow incoming TCP packets on port 2200 to allow SSH: 
`sudo ufw allow 2200/tcp`
4. Allow incoming TCP packets on port 80 to allow www: 
`sudo ufw allow www`
5. Allow incoming UDP packets on port 123 to allow NTP: 
`sudo ufw allow 123/udp`
6. Close port 22: 
`sudo ufw deny 22`
7. Enable firewall: 
`sudo ufw enable`
8. Check out current firewall status: 
`sudo ufw status`

Now firewall has been configured. Need to login and  check if 22 is closed and 2200 is enabled.
for that you can `ssh localhost -p 2200` -p stands for port.
- For port 22 the connection has to refuse. 
- For the port 2200 the connection should not refuse.
If this scenario works then we can precede further To login to 52.15.135.223 as grader



## Login to server as grader

we need the following for loging

#### grader_dev.key
/*
I have removed the rsa key as i have completed the nano degree
*/

 > grader_dev passphrase : ******

`sudo ssh -i grader_dev.key grader@52.15.135.223 -p 2200`
prompts for password, enter the pass phrase

>grader@ip-172-**-*-**1:~$

## Configuring Web Server
Installing Apache
`sudo apt-get install apache2`

Testing Apache using the IP address shows the default Apache homepage.
http://52.15.135.223/ or http://ec2-52-15-135-223.us-east-2.compute.amazonaws.com/

#### Install python3 and  configure mod_wsgi package
The server had inbuilt python2.7 and had to install python3 and pip3
`sudo apt-get install python3`
`sudo apt-get install python3-pip`
> to check if python is working fine at the prompt type `python3 -V`
and `pip3 -V`

Now its time to git checkout your project. Once you checked out the project files we need to install project dependencies.
git clone https://github.com/vinayrajan/itemcatalog-serverconfig.git to destination folder

`sudo pip3 -r requirements.txt` 

#### mod_wsgi
1. Install the mod_wsgi package that will communicate between apache2 and python : 
`sudo apt-get install python3-dev`
`sudo apt-get install libapache2-mod-wsgi-py3`
2. Enable mod_wsgi on apache2: 
`sudo a2enmod wsgi`
3. Restart Apache: 
`sudo service apache2 restart``

## Installing Database
Installed  PostgreSQL and configured using the following commands
`sudo apt-get install postgresql`
Make sure PostgreSQL does not allow remote connections
my PostgreSQL version was 10.  
Create a new database and user named `catalog` with password  and set permissions to the database database.
`sudo nano /etc/postgresql/10/main/pg_hba.conf`

>check for these lines it needs to be exactly like below
```
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
```
#### Create new PostgreSQL user called catalog
login to PostgreSQL 
`sudo su - postgres` Type in `psql` as PostgreSQL user
```
CREATE USER catalog WITH PASSWORD 'catalog';
```
> CREATE ROLE
```
postgres=# ALTER USER catalog CREATEDB;
```
> ALTER ROLE
```
postgres=# CREATE DATABASE catalog WITH OWNER catalog;
```
>CREATE DATABASE

Now we have created a user called *catalog* with password *catalog* gave user role, and created a database called *catalog* for which the owner is the user *catalog*
>to check if database has been created 
`postgres=# \c catalog
You are now connected to database "catalog" as user "postgres".`

to exit out of PostgreSQL type `\q` and  `exit`

## Configuring Website
Created a config for apache2 on `/etc/apache2/sites-avilable/catalog.conf` for the requests to redirect to the item catalog project
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
enabled the configuration
`sudo a2ensite catalog.conf`


## Testing
Installed Lynx browser to test the site internally.
Changed the **OAuth 2.0 client ID** from  console.developers.google.com


# Website Urls
Apache default page : http://52.15.135.223/
or http://ec2-52-15-135-223.us-east-2.compute.amazonaws.com

#### Home Page
http://ec2-52-15-135-223.us-east-2.compute.amazonaws.com/catalog/

#### Json Endpoints
http://ec2-52-15-135-223.us-east-2.compute.amazonaws.com/catalog/products/json

http://ec2-52-15-135-223.us-east-2.compute.amazonaws.com/catalog/product/1/json

http://ec2-52-15-135-223.us-east-2.compute.amazonaws.com/catalog/categories/json

http://ec2-52-15-135-223.us-east-2.compute.amazonaws.com/catalog/category/1/json


    
Thankyou
Vinay Kumar Rajan
vr3924@intl.att.com
