# Linux Server Configuration and Deployment of the Item Catalog App

## Access

- public IP >>> 18.185.188.91
- SSH port >>> 2200
- url >>> http://18.185.188.91.xip.io/

## Packages installed

- apache2
- libapache2-mod-wsgi
- postgresql
- python-pip
- python-flask 
- python-sqlalchemy 
- python-psycopg2
- oauth2client 
- requests 
- httplib2

## Configuration summary

1. Initial SSH and update:
   - `ssh ubuntu@publicIP -p 22 -i ~/.ssh/privateKey`
   - `sudo apt-get update`
   - `sudo apt-get upgrade`

2. The `grader` user:
   - `sudo adduser grader`
   - `sudo nano /etc/sudoers.d/grader` (grader ALL=(ALL) NOPASSWD:ALL)
   - `mkdir /home/grader/.ssh`
   - `nano /home/grader/.ssh/authorized_keys` (public key generated locally)
   - `chmod 700 home/grader/.ssh`
   - `chmod 644 home/grader/.ssh/authorized_keys`

3. Ports, firewall, timezone:
   - `sudo nano /etc/ssh/sshd_config` (change port 22 to 2200)
   - `sudo service ssh restart`
   - `sudo ufw default deny incoming`
   - `sudo ufw default allow outgoing`
   - `sudo ufw allow 2200/tcp`
   - `sudo ufw allow www`
   - `sudo ufw allow ntp`
   - `sudo ufw enable` (check with logout and new ssh on -p 2200)
   - `sudo dpkg-reconfigure tzdata` (choose none, then UTC)

4. Apache, WSGI, Git:
   - `sudo apt-get install apache2`
   - `sudo apt-get install libapache2-mod-wsgi python-dev` 
     (check with `sudo a2enmod wsgi`)
   - `sudo mkdir /var/www/catalog` (and change there)
   - `sudo git clone https://github.com/mostrovski/ChessReads.git catalog`
   - `sudo nano /var/www/catalog/catalog/catalog.wsgi` (WSGI File)
   - `sudo nano /etc/apache2/sites-available/catalog.conf` (Virtual Host File)
   
   *WSGI*
   ```
   import sys
   import logging

   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0, '/var/www/catalog/catalog')

   from app import app as application
   application.secret_key='super_secret_key'
   ```
   *Virtual Host*
   ```
   <VirtualHost *:80>
       ServerName  publicIP
       ServerAdmin my email
       WSGIScriptAlias / /var/www/catalog/catalog/catalog.wsgi
       <Directory /var/www/catalog/catalog/>
            Order allow,deny
            Allow from all
       </Directory>
       Alias /static /var/www/FlaskApp/ItemCatalog/static
       <Directory /var/www/catalog/catalog/static/>
          Order allow,deny
          Allow from all
       </Directory>
       ErrorLog ${APACHE_LOG_DIR}/error.log
       LogLevel warn
       CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```
5. Flask and libraries:
   - `sudo apt-get install python-pip python-flask`
   - `sudo apt-get install python-sqlalchemy python-psycopg2`
   - `sudo pip install oauth2client requests httplib2`

6. Main changes to the application files:
   - create_engine is everywhere like 
     `create_engine('postgresql://dbuser:password@localhost/dbname')`
   - path to client_secrets.json includes `/var/www/catalog/catalog/`
   - `app.secret_key` and `app.debug` are removed
   - `app.run()` is left without arguments 

7. PostgreSQL, database setup and populate:
   - `sudo apt-get install postgresql` 
     (check /etc/postgresql/9.5/main/pg_hba.conf)
   - `sudo -u postgres psql`
     (CREATE USER username WITH PASSWORD password;
      CREATE DATABASE databasename;)
   - `cd /var/www/catalog/catalog`
   - `sudo python db_setup.py`
   - `sudo python db_populate.py`

## Resources

http://flask.pocoo.org/docs/1.0/tutorial/deploy/
http://flask.pocoo.org/docs/1.0/tutorial/deploy/
https://forums.aws.amazon.com/thread.jspa?threadID=244112
https://medium.com/@zionoyemade/deploying-flask-application-to-amazon-lightsail-199e79bb256a
https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-xvii-deployment-on-linux
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
https://github.com/sarithk/LinuxServerConfig
https://github.com/harushimo/linux-server-configuration