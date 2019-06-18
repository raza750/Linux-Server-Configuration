# Linux-Server-Configuration
<h3>Project Overview</h3>
<p>A baseline installation of a Linux server and prepare it to host web applications. Learning how to secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.</p>
<div>
Public IP address: 13.232.26.143</br>
SSH port: 2200</br>
URL: http://13.232.26.143/ </br>
</div>
<h5>1. Steps to Configure Linux server</h5>
<p>Start a new Ubuntu Linux server instance on <a target="_blank" href="https://lightsail.aws.amazon.com">Amazon Lightsail</a>.
You can refer to the <a href="https://aws.amazon.com/documentation/lightsail/" rel="nofollow">documentation</a> which will help you to get started.</p>
<h6>Follow the instructions provided to SSH into your server.</h6>
<p>There is a button on lightsail dashboard to directly SSH into your server. You can also SSH into your machine using the private key.</p>
<p>Download the private key provided in account section of AWS Lightsail.</p>
<p>Use this command: $ ssh -i <privateKeyOfInstance.rsa> <Username>@<Public IP address></p>


<h5>3. Update all currently installed packages</h5>
</p>sudo apt-get update</p>
</p>sudo apt-get upgrade</p>


<h5>4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.</h5>
<p>sudo vim /etc/ssh/sshd_config</p>
<p>change the Port 22 to 2200</p>
save and exit</br>
restart ssh service
<p>sudo service ssh restart</p>


Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
sudo ufw status                  
sudo ufw default deny incoming   
sudo ufw default allow outgoing  
sudo ufw allow 2200/tcp          
sudo ufw allow www               
sudo ufw allow 123/udp           
sudo ufw deny 22  
sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup

Create a new user account named grader.
sudo adduser grader
sudo vim /etc/sudoers.d/grader
Edit and following line to this file
grader ALL=(ALL) NOPASSWD:ALL

Verify that grader has sudo permissions. 
su - grader 
enter the password 
sudo -l 
enter the password again. The output should be like this:
Matching Defaults entries for grader on
    ip-172-26-5-148.ap-south-1.compute.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User grader may run the following commands on
        ip-172-26-5-148.ap-south-1.compute.internal:
    (ALL) NOPASSWD: ALL

	
Create an SSH key pair for grader using the ssh-keygen tool
Generate a keypair and push it to server. Use your local machine to generate a key pair
$ssh-keygen 
Push it to server: Create .ssh directory on server machine. And follow the commands to push and authorize the key for SSH login.
sudo mkdir ~/.ssh
sudo touch ~/.ssh/authorized_keys
Copy and paste the key from your local machine, usign vim editor:
sudo vim ~/.ssh/authorized_keys
Changing permission of .ssh and .ssh/authorized_keys
sudo chmod 700 .ssh
sudo chmod 644 .ssh/authorized_keys
Check in /etc/ssh/sshd_config file if PasswordAuthentication is set to no
Restart SSH: sudo service ssh restart
On the local machine
ssh -i udacity_key grader@13.232.26.143 -p 2200


Prepare to deploy your project.
Configure the local timezone to UTC.
sudo timedatectl set-timezone UTC

While logged in as grader, Install and configure Apache to serve a Python mod_wsgi application.
sudo apt-get install apache2 
sudo apt-get install libapache2-mod-wsgi

Enable mod_wsgi using: 
sudo a2enmod wsgi

Installing Postgresql python dependencies
sudo apt-get install libpq-dev 
sudo apt-get install python-dev

Install PostgreSQL: 
sudo apt-get install postgresql
sudo apt-get install postgresql-contrib

PostgreSQL should not allow remote connections. In the /etc/postgresql/9.5/main/pg_hba.conf file, you should see:
sudo cat /etc/postgresql/9.5/main/pg_hba.conf
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
#local   replication     postgres                                peer
#host    replication     postgres        127.0.0.1/32            md5
#host    replication     postgres        ::1/128                 md5


Create a new database user named catalog that has limited permissions to your catalog application database.
sudo su - postgres
psql
Create a new database named catalog:  CREATE DATABASE catalog;
Create a new user named catalog:  CREATE USER catalog;
Set a password for catalog user:  ALTER ROLE catalog with password 'password';
Grant permission to catalog user:  GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
Exit from psql:  \q;
Return to grader using: $ exit


Install python-pip, Flask and other dependencies
sudo apt-get install python3-pip
pip install httplib2
pip install requests
pip install oauth2client
sudo -H pip install sqlalchemy
sudo -H pip install flask
sudo apt-get install libpq-dev
pip install psycopg2



Install git
sudo apt-get install git

Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.
Make a udacity named directory in /var/www/ and udacity in udacity
sudo mkdir /var/www/udacity
Make grader as ownner of that directory
sudo chown -R grader:grader /var/www/udacity
Clone the Item Catalog and put them in the /var/www/udacity directory:
sudo git clone https://github.com/raza750/Catalog_Udacity
change the name of Catalog_Udacity to udacity
sudo mv Catalog_Udacity udacity
Change the name of appone.py to __init__.py:
sudo mv appone.py __init__.py

Change the database connection in database.py, dummyData.py and __init__.py to:
engine = create_engine('postgresql://catalog:password@localhost/catalog')

In __init__ change app.run(host='0.0.0.0', port=5000) to app.run() in __main__ method and change the path of the client id:
CLIENT_ID = json.loads(open('/var/www/udacity/udacity/client_secrets.json', 'r').read())['web']['client_id']

Create the .wsgi file in /var/www/udacity 
$ cd /var/www/udacity/
$ sudo vim udacity.wsgi
Add the following lines of code to the .wsgi file

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/udacity/")

from udacity import app as application
application.secret_key = 'my_secret_key'


Configure and Enable a New Virtual Host:
sudo vim /etc/apache2/sites-available/udacity.conf

Add the following lines of code to the file to configure the virtual host
<VirtualHost *:80>
	ServerName 13.232.26.143
	ServerAdmin grader@13.232.26.143
	WSGIScriptAlias / /var/www/udacity/udacity.wsgi
	<Directory /var/www/udacity/udacity/>
		Order allow,deny
		Allow from all
	</Directory>
	
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

Enable the virtual host with the following command:
sudo a2ensite udacity.conf
Restart Apache to run the app on sever
sudo service apache2 restart

for server erros use :
sudo cat /var/log/apache2/error.log
