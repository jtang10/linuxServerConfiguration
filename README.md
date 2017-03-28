# Linux Server Configuration
##
### Overview

This is the 7th project in Udacity Full Stack Web Developer Nanodegree. The objective is to deploy a linux server using Amazon AWS Lightsail and serve the 5th project item catalog.

### Useful Info

IP address: 107.21.43.58

URL: http://ec2-107-21-43-58.compute-1.amazonaws.com

SSH port: 2200

### Setup
1. **Register an AWS account** on [Lightsail](http://lightsail.aws.amazon.com) and create an instance. Follow the Udacity tutorial to perform this.
2. In the Lightsail page's Networking tab, open the port 2200 in Firewall and create a static-IP for the instance. 
3. Download the default key pair from Lightsail so that you can login the server by `$ ssh -i PrivateKey ubuntu@YourIP`.
4. **change the SSH port from 22 to 2200**. 
	* Open the ssh config file by `$ sudo nano /etc/ssh/sshd_config`. Find the **Port** line and change it from 22 to 2200
	* Set `PermitRootLogin no`. Also enforce the key-based authentication by setting `PasswordAuthentication no`.
	* Restart the ssh by `$ sudo service ssh restart`. 
	* From now on, you have to login from port current SSH port 2200 by `$ ssh -i PrivateKey ubuntu@YourIP -p 2200`.
5. **Create new user "grader" and grant sudo permissions** with
	* `$ sudo addruser grader`
	* `$ sudo nano /etc/sudoers.d/grader`. Add the following line "grader ALL=(ALL:ALL) ALL".
6. Generate the encryption key by
	* Gernerate the encryption key **in your local machine** with `$ ssh-keygen ~/.ssh/udacity_key`.
	* Login the server and copy the key here by `$ touch /home/grader/.ssh/authorized_keys`
	* Copy the content of public key *udacity_key.pub* from the local machine. Change the permission and ownership of the key byperform
		- `$ sudo chmod 700 /home/grader/.ssh`.
		- `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`.
		- `$ sudo chown -R grader:grader /home/grader/.ssh`.
7. From now, you can use the grader's key to login the server with user *grader*.
8. **Configure the UFW of the server** by 
	* `$ sudo ufw allow 2200/tcp`.
	* `$ sudo ufw allow 80/tcp`.
	* `$ sudo uufw allow 123/udp`.
	* `$ sudo ufw enable`.
9. **Install Apache and mod_wsgi** and update all the packages
	* `$ sudo apt-get update`
	* `$ sudo apt-get install apache2`
	* `$ sudo apt-get install libapache2-mod-wsgi`
10. **Install Git and python**. This should be already installed in the Lightsail server but you can still check that.
11. **Clone the itemCatalog app from GitHub**. 
	* `$ sudo mkdir /var/www/itemCatalog`
	* Change the ownership of the created directory `$ sudo chown -R grader:grader itemCatalog`.
	* Move inside the the itemCatalog directory `$ cd /var/www/itemCatalog/` and clone the item catalog repository `git clone https://github.com/jtang10/itemCatalog.git`
	* Create a *itemCatalog.wsgi* file to serve the applciation by pasting the following code:
      ```
      import sys
      import logging
      logging.basicConfig(stream=sys.stderr)
      sys.path.insert(0, "/var/www/itemCatalog/itemCatalog")
      from itemCatalog import app as application
      ```
12. **Install the virtual environment**
	* `$ sudo apt-get install python-pip`
	* `$ sudo pip install virtualenv`
	* Create a virtual environment for this project by `$ sudo virtualenv venv`.
	* Activate the virtual environment by `$ source venv/bin/activate`.
	* Change the permission of the virtual environment folder `$ sudo chmod -R 777 venv`.
	* Install all the required modules including Flask, httplib2, requests, oauth2client, sqlalchemy and psycopg2.
13. **Configure and enable a new virtual host**
	* Create a virtual host config file `$ sudo nano /etc/apache2/sites-available/itemCatalog.conf`.
	* Paste the following code:
	  ```
      <VirtualHost *:80>
          ServerName 107.21.43.58
          ServerAlias http://ec2-107-21-43-58.compute-1.amazonaws.com
          ServerAdmin admin@107.21.43.58
          WSGIDaemonProcess itemCatalog python-path=/var/www/catalog:/var/www/itemCatalog/venv/lib/python2.7/site-packages
          WSGIProcessGroup itemCatalog
          WSGIScriptAlias / /var/www/itemCatalog/itemCatalog.wsgi
          <Directory /var/www/itemCatalog/itemCatalog/>
              Order allow,deny
              Allow from all
          </Directory>
          Alias /static /var/www/itemCatalog/itemCatalog/static
          <Directory /var/www/itemCatalog/itemCatalog/static/>
              Order allow,deny
              Allow from all
          </Directory>
          ErrorLog ${APACHE_LOG_DIR}/error.log
          LogLevel warn
          CustomLog ${APACHE_LOG_DIR}/access.log combined
      </VirtualHost>
      ```
     * Enable the new virtual host `$ sudo a2ensite itemCatalog`.
14. **Install and configure PostgreSQL**
	* `$ sudo apt-get install postgresql`.
	* There is an automatically created user for PostgreSQL called *postgres". You can use it to login the SQL by `$ sudo su - postgres` and then `$ psql`.
	* Create a new user `# CREATE USER itemcatalog WITH PASSWORD 'password';`
	* Give *itemcatalog* the CREATEDB capability: `# ALTER USER itemcatalog CREATEDB;`
	* Create a database *itemcatalog* owned by user *itemcatalog*: `# CREATE DATABASE itemcatalog WITH OWNER catalog;`
	* Connect to the database `# \c itemcatalog`.
	* Revoke all rights: `# REVOKE ALL ON SCHEMA public FROM public;`
	* Let only *itemcatalog* create tables: `# GRANT ALL ON SCHEMA public TO itemcatalog;`
	* Log out from PostgreSQL: `# \q` and then return to the terminal: `$ exit`.
	* The database in the Flask app would now be connected by 
		
        `engine = create_engine('postgresql://itemcatalog:password@localhost/itemcatalog')`
    * Setup the database and fill them with some examples by `$ python database_setup.py` and then `$ python lotsofitems.py`
    
15. Restart the apache server to serve the web application `$ sudo service apache2 restart`.


	
    
