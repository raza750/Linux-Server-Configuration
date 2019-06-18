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
<h5>2. Follow the instructions provided to SSH into your server.</h5>
<p>There is a button on lightsail dashboard to directly SSH into your server. You can also SSH into your machine using the private key.</p>
<h6>Download the private key provided in account section of AWS Lightsail.</h6>
<p>Use this command: $ ssh -i LightsailDefaultKey-ap-south-1.pem grader@13.232.26.143</p>


<h5>3. Update all currently installed packages</h5>
</p>sudo apt-get update</p>
</p>sudo apt-get upgrade</p>


<h5>4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.</h5>
<p>sudo vim /etc/ssh/sshd_config</p>
<p>change the Port 22 to 2200</p>
<h8>save and exit</h8>
<h8>restart ssh service</h8>
<p>sudo service ssh restart</p>


<h5>5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).</h5>

<p>sudo ufw status</p>                  
<p>sudo ufw default deny incoming</p>   
<p>sudo ufw default allow outgoing</p>  
<p>sudo ufw allow 2200/tcp</p>          
<p>sudo ufw allow www</p>               
<p>sudo ufw allow 123/udp</p>           
<p>sudo ufw deny 22</p>  
<p>sudo ufw enable</p>
<p>Command may disrupt existing ssh connections. Proceed with operation (y|n)? y</p>
<p>Firewall is active and enabled on system startup</p>

<h6>Click on the Manage option of the Amazon Lightsail Instance, then the Networking tab, and then change the firewall configuration to match the internal firewall settings above. </h6>

<h5>6. Create a new user account named grader.</h5>
<p>sudo adduser grader</p>
<p>sudo vim /etc/sudoers.d/grader</p>
<h6>Edit and add following line to this file</h6>
<p>grader ALL=(ALL) NOPASSWD:ALL</p>

<p>Verify that grader has sudo permissions.</br>
<p>su - grader </p>
<h6>enter the password</h6> 
<p>sudo -l </p>
<h6>enter the password again. The output should be like this:</6>
Matching Defaults entries for grader on
    ip-172-26-5-148.ap-south-1.compute.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User grader may run the following commands on
        ip-172-26-5-148.ap-south-1.compute.internal:
    (ALL) NOPASSWD: ALL
    </p>

	
<h5>7. Create an SSH key pair for grader using the ssh-keygen tool.</h5>
<h6>Generate a keypair and push it to server. Use your local machine to generate a key pair</h6>
<p>$ssh-keygen </p>
<h6>Push it to server: Create .ssh directory on server machine. And follow the commands to push and authorize the key for SSH login.</h6>
<p>sudo mkdir ~/.ssh</p>
<p>sudo touch ~/.ssh/authorized_keys</p>
<h6>Copy and paste the key from your local machine, usign vim editor:</h6>
<p>sudo vim ~/.ssh/authorized_keys</p>
<h6>Changing permission of .ssh and .ssh/authorized_keys</h6>
<p>sudo chmod 700 .ssh</p>
<p>sudo chmod 644 .ssh/authorized_keys</p>
<h7>Check in /etc/ssh/sshd_config file if PasswordAuthentication is set to no</h7>
<p>Restart SSH: sudo service ssh restart</p>
<h6>On the local machine</h6>
<p>ssh -i udacity_key grader@13.232.26.143 -p 2200</p>



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
