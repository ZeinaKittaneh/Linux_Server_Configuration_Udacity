# Linux-Server-Configuration Udacity Full-Stack-NanoDegree Project
This project is the fifth project of FSND, it aims to configure a linux server to host the itemCatalog App which was created earlier in this Nano Degree:

## Server details:
server used is an Amazon Lightsail server, with Ubuntu `18.04` OS.
public server IP address: http://35.183.105.8 
The database server is PostgreSQL.

## Main Tasks:
1. Configure Ubuntu Linux server instance on Amazon Lightsail.
2. Secure the server: Update all currently installed packages, change ssh port and configure firewall
3. Give grader access.
4. Configure the local timezone to UTC.
5. Install and configure Apache to serve a Python mod_wsgi application.
6. Install and configure PostgreSQL and do not allow remote connections
7. Install git.
8. Deploy the Item Catalog project.

## 1. Server Configuration:
* Create a new Ubuntu Linux server instance on Amazon Lighsail:
    https://lightsail.aws.amazon.com/
* Configure ssh access to the instance:
    1. Download the private key: from the amazon lightsail "account" menu click the SSH keys to download the key.
    2. Copy the key from the file you downloaded, then create a file in the ` ~/.ssh `, call it serverKey.rsa and paste the private key there.
    3. change the permissions of the key using:
    `chmod 600 ~/.ssh/serverKey.rsa`
    4. Connect to the instance via:
    `ssh -i ~/.ssh/serverKey.rsa ubuntu@35.183.105.8 where 35.183.105.8 is the public IP address` of the instance.

## 2. Secure the server:
1) Update all currently installed packages:
    ```
    sudo apt-get update
    sudo apt-get upgrade
    ```
2) Change the SSH port from 22 to 2200. (Make sure to configure the Lightsail firewall to allow it).
    * Edit the /etc/ssh/sshd_config file via: `sudo nano /etc/ssh/sshd_config`.
    * Change the port number on line 5 from 22 to 2200 then Save and exit using CTRL+X and confirm with Y.
    * Restart SSH via: `sudo service ssh restart.`

3) Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

    * we need to deny all incoming traffic and allow only on (2200, 80, 123) ports.
    ```
    sudo ufw status                  # The UFW should be inactive.
    sudo ufw default deny incoming   # Deny any incoming traffic.
    sudo ufw default allow outgoing  # Enable outgoing traffic.
    sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
    sudo ufw allow www               # Allow HTTP traffic in.
    sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
    sudo ufw deny 22                 # Deny tcp and udp packets on port 53.
    ```
    * Then we need to turn on the UFW: `sudo ufw enable`.
    * Finally, check the status of the firewall: `sudo ufw status`, the output should show the ports we allowed and denied.
    * From the Amazon Lightsail Instance page, got to the Networking tab, change the configuration to match our firewall settings we've made before.

## 3. Create grader user with sudo permissions:
1) Create a new user called "grader":
`sudo adduser grader`
then enter the password.

2) Give the "grader" user sudo permissions:
    * Edit the sudoers file: `sudo visudo`
    * Search for this line: `root    ALL=(ALL:ALL) ALL`
    * then type in: `grader ALL=(ALL:ALL) ALL`, save and quit
    * you can verify the sudo permissions of the "grader" user by running: `su - grader` then enter the password then run `sudo -l`.

3) Login using ssh keys:
    generate ssh keys on local machine using: `ssh-keygen`, choose a file name (ex. grader_key) ; then save the private key in `~/.ssh on local machine`.
    Run `cat ~/.ssh/grader_key.pub` and copy the contents of the file.

4) Then on the virtual machine:
    ```
     su - grader
     mkdir .ssh
     touch .ssh/authorized_keys
     vim .ssh/authorized_keys
     ```
    Paste the public key generated on the local machine into this file and save

5) We need then to change the permissions of the ssh files:
    ```
     chmod 700 .ssh
     chmod 644 .ssh/authorized_keys
     ```
    restart the SSH via: `service ssh restart`

6) To login using the SSH Key run the following command:
`ssh -i ~/.ssh/grader_key -p 2200 grader@35.183.105.8`

## 4. Configure the local timezone to UTC.
While logged in as "grader" user, run the following command : `sudo dpkg-reconfigure tzdata`
the output should be something like:
```
Current default time zone: 'America/Montreal'
Local time is now:      Mon Apr 22 22:21:52 EDT 2019.
Universal Time is now:  Tue Apr 23 02:21:52 UTC 2019.
```

## 5. Install and configure Apache to serve a Python mod_wsgi application.
* While logged in as "grader" user, run `sudo apt-get install apache2` to install Apache.
* Install python3(depending on the python version you use) via: `sudo apt-get install libapache2-mod-wsgi-py3`.
* Enable mod_wsgi using: `sudo a2enmod wsgi`.

## 6. Install and configure PostgreSQL:
* While logged in as grader, install PostgreSQL: `sudo apt-get install postgresql`
* Then check if remote connections are not allowed using: `vim /etc/postgresql/9.5/main/pg_hba.conf`

* To prepare for the deployment of the ItemCatalog project, we need to create a user (ex. catalog) and give him the ability to create databases, follow these steps:

1. Switch to the postgres user: `sudo su - postgres`.

2. Open PostgreSQL interactive terminal with `psql`.

3. Create the catalog user with a password and give him the ability to create databases:
    * Run the following commands:
    ```
    CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
    ALTER ROLE catalog CREATEDB`;
    ```
    * To verify, list the existing roles: `\du`.

    * Switch back to the grader user: `exit`.
    
    * Create a new Linux user called catalog: `sudo adduser catalog`.
    
    * Give to catalog user the permission to sudo. Run: `sudo visudo`.

        * Search for this line: `grader  ALL=(ALL:ALL) ALL`
        * Below this line, add the following line to give sudo privileges to catalog user.
        `catalog  ALL=(ALL:ALL) ALL`
        
        * Save and exit using CTRL+X and confirm with Y.

    * Verify that catalog has sudo permissions: Run `su - catalog`, then run `sudo -l` 
    
    * While logged in as catalog, create a database: `createdb catalog`.

    * Run `psql` and then run `\l` to see that the new database has been created. 
    * Exit psql: `\q` then switch back to the grader user: `exit`.

## 7. Install git.
Login as "grader" user, then install git using:`sudo apt-get install git`.

## 8. Deploy the Item Catalog project.
1. Clone the Item Catalog project from the GitHub repository:
* Login as "grader" user then create a directory to store the project, call it "catalog":
`mkdir /var/www/catalog`

* Go to this directory: `cd /var/www/catalog` and clone the project from GitHub:
`sudo git clone https://github.com/ZeinaKittaneh/Item_Catalog_Udacity.git itemCatalog`

2. Setup the ItemCatalog project:
* Go to `/var/www` and change the ownership of the catalog directory to "grader":
`sudo chown -R grader:grader catalog/`

* Go to the `/var/www/catalog/itemCatalog` directory.

* Rename the catalog.py file to __init__.py using: `mv catalog.py __init__.py`.

* In __init__.py, replace this line:
`app.run(host="0.0.0.0", port=8000, debug=True)`
by: `app.run()`

* In database.py, replace this line:
`engine = create_engine("sqlite:///catalog.db")`
by: `engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')`

3. Install the virtual environment and dependencies:
    * Login as "grader" user, install pip: `sudo apt-get install python3-pip`.
    
    * Then Install the virtual environment using: `sudo apt-get install python-virtualenv`

    * Go to the `/var/www/catalog/catalog/` directory.
    
    * Create the virtual environment: `sudo virtualenv -p python3 venv3`.
    
    * Change the ownership to grader with: `sudo chown -R grader:grader venv3/`.
    
    * Activate the new environment: `. venv3/bin/activate`.
    
    * Install the following dependencies:
        ```
        pip install httplib2
        pip install requests
        pip install --upgrade oauth2client
        pip install sqlalchemy
        pip install flask
        sudo apt-get install libpq-dev
        pip install psycopg2
        ```
    * Run python3 __init__.py and you should see:
    `Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)` 
    
4. Enable the virtual host:
    * To enable python3 version, go to : `/etc/apache2/mods-enabled/wsgi.conf`
    and add: WSGIPythonPath `/var/www/catalog/itemcatalog/venv3/lib/python3.6.7/site-packages`
    
    * Create `/etc/apache2/sites-available/catalog.conf` and add the following lines to configure the virtual host:
        ```
        <VirtualHost *:80>
            ServerName 99.79.72.192
            ServerAdmin zeinakittaneh@yahoo.com
            WSGIScriptAlias / /var/www/catalog/catalog.wsgi
            <Directory /var/www/catalog/itemcatalog/>
                Order allow,deny
                  Allow from all
            </Directory>
            Alias /static /var/www/catalog/itemcatalog/static
            <Directory /var/www/catalog/itemcatalog/static/>
                  Order allow,deny
                  Allow from all
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>
        ```
    * Enable virtual host using: `sudo a2ensite itemCatalog`
    * Reload Apache: `sudo service apache2 reload`.

5. Setup the Flask application:
    * Create `/var/www/catalog/catalog.wsgi` file add the following lines:
        ```
        activate_this = '/var/www/catalog/itemcatalog/venv3/bin/activate_this.py'
        with open(activate_this) as file_:
            exec(file_.read(), dict(__file__=activate_this))
    
        #!/usr/bin/python
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/catalog/itemcatalog")
        sys.path.insert(1, "/var/www/catalog/")
        
        from itemcatalog import app as application
        application.secret_key = 'super secret key'
        ```
    * Restart Apache: `sudo service apache2 restart`.

6. Setup and populate the database:
    * Go to: /var/www/catalog/itemCatalog/database_setup.py and the these lines at the beginning of the file.
        ```
        import sys
        sys.path.insert(0, "/var/www/catalog/catalog/venv3/lib/python3.5/site-packages") 
        ```
    * From the /var/www/catalog/itemCatalog/ directory, activate the virtual environment:
    `. venv3/bin/activate`.
    
    * Run: python `lotsOfCategories.py`.
    
    * Deactivate the virtual environment: `deactivate`.

7. Disable the default Apache site:
run the following command: `sudo a2dissite 000-default.conf`

8. Launch the Web Application
    * Change the ownership of the project directories using:
    `sudo chown -R www-data:www-data itemCatalog/`.
    * Restart Apache again: `sudo service apache2 restart`.
    * Open your browser to http://35.183.105.8

## Helpful Resources:
https://serverpilot.io/docs/how-to-create-a-server-on-amazon-lightsail
https://www.digitalocean.com/community/tutorials/how-to-serve-django-applications-with-apache-and-mod_wsgi-on-ubuntu-16-04
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-18-04
http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/#working-with-virtual-environments
https://vitux.com/install-python3-on-ubuntu-and-set-up-a-virtual-programming-environment/
