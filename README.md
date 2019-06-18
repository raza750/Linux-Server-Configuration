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


<h4>8. Configure the local timezone to UTC.</h4>
<p>sudo timedatectl set-timezone UTC</p>

<h4>9. While logged in as grader, Install and configure Apache to serve a Python mod_wsgi application.</h4>
<p>sudo apt-get install apache2 </p>
<p>sudo apt-get install libapache2-mod-wsgi </p>

<h6>Enable mod_wsgi using: </h6>
<p>sudo a2enmod wsgi</p>

<h4>10. Installing Postgresql python dependencies</h4>
<p>sudo apt-get install libpq-dev</p> 
<p>sudo apt-get install python-dev</p>

<h6>Install PostgreSQL:<h6> 
<p>sudo apt-get install postgresql</p>
<p>sudo apt-get install postgresql-contrib</p>

<h5>PostgreSQL should not allow remote connections. In the /etc/postgresql/9.5/main/pg_hba.conf file, you should see:</h5>
<p>sudo cat /etc/postgresql/9.5/main/pg_hba.conf</p>
<p>local   all             postgres                                peer

 TYPE  DATABASE        USER            ADDRESS                 METHOD

 "local" is for Unix domain socket connections only
local   all             all                                     peer
 IPv4 local connections:
host    all             all             127.0.0.1/32            md5
 IPv6 local connections:
host    all             all             ::1/128                 md5
 Allow replication connections from localhost, by a user with the
 replication privilege.
#local   replication     postgres                                peer
#host    replication     postgres        127.0.0.1/32            md5
#host    replication     postgres        ::1/128                 md5</p>


<h4>11. Create a new database user named catalog that has limited permissions to your catalog application database.</h4>
<p>sudo su - postgres</p>
<p>psql</p>
<h6>Create a new database named catalog:  CREATE DATABASE catalog;</h6>
<h6>Create a new user named catalog:  CREATE USER catalog;</h6>
<h6>Set a password for catalog user:  ALTER ROLE catalog with password 'password';</h6>
<h6>Grant permission to catalog user:  GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;</h6>
<h6>Exit from psql:  \q;</h6>
<h6>Return to grader using: $ exit</h6>


<h4>12. Install python-pip, Flask and other dependencies<h4>
<p>sudo apt-get install python-pip</p>
<p>pip install httplib2</p>
<p>pip install requests</p>
<p>pip install oauth2client</p>
<p>sudo -H pip install sqlalchemy</p>
<p>sudo -H pip install flask</p>
<p>sudo apt-get install libpq-dev</p>
<p>pip install psycopg2</p>


<h4>13. Install git</h4>
<p>sudo apt-get install git</p>

<h5>Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.</h5>

<h6>Make a udacity named directory in /var/www/ and udacity in udacity</h6>
<p>sudo mkdir /var/www/udacity</p>
<h6>Make grader as ownner of that directory</h6>
<p>sudo chown -R grader:grader /var/www/udacity</p>
<h5>Clone the Item Catalog and put them in the /var/www/udacity directory:</h5>
<p>sudo git clone https://github.com/raza750/Catalog_Udacity</p>
<h6>change the name of Catalog_Udacity to udacity<h6>
<p>sudo mv Catalog_Udacity udacity</p>
<h6>Change the name of appone.py to __init__.py:</h6>
<p>sudo mv appone.py __init__.py</p>

<h4>14. Change the database connection in database.py, dummyData.py and __init__.py to:</h4>
<p>engine = create_engine('postgresql://catalog:password@localhost/catalog')</p>

<h6>In __init__ change app.run(host='0.0.0.0', port=5000) to app.run() in __main__ method and change the path of the client id:
CLIENT_ID = json.loads(open('/var/www/udacity/udacity/client_secrets.json', 'r').read())['web']['client_id'] </h6>


<h4>15. Create the .wsgi file in /var/www/udacity </h4>
<p>cd /var/www/udacity/</p>
<p>sudo vim udacity.wsgi</p>

<h6>Add the following lines of code to the .wsgi file</h6>
<p>
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/udacity/")

from udacity import app as application
application.secret_key = 'my_secret_key'
</p>


<h4>16. Configure and Enable a New Virtual Host:</h4>
<p>sudo vim /etc/apache2/sites-available/udacity.conf</p>

<h6>Add the following lines of code to the file to configure the virtual host</h6>
<p>
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
</p>

<h4>17. Enable the virtual host with the following command</h4>
<p>sudo a2ensite udacity.conf</p>
<h6>Restart Apache to run the app on sever</h6>
<p>sudo service apache2 restart</p>

<h6>for server erros use </h6>
<p>sudo cat /var/log/apache2/error.log</p>
