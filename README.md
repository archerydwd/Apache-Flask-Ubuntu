# Apache-Flask-Ubuntu
Instructions on installing Apache &amp; Flask on ubuntu

=
###Preconditions

> You have a clean instance of ubuntu running.

You can get ubuntu from here: http://www.ubuntu.com/download

=
###Install Apache

Open a terminal and enter the following commands:

```
sudo apt-get upgrade
sudo apt-get update
sudo apt-get install apache2
sudo apt-get install vim        //or your favourite text editor
```

=
###Install Python

**On OSX** 

Follow the link: https://www.python.org/ftp/python/3.4.3/python-3.4.3-macosx10.6.pkg
Open this and follow the instructions.

**On Linux**

```
wget http://www.python.org/ftp/python/3.4.3/Python-3.4.3.tgz
tar -xzf Python-3.4.3.tgz  
cd Python-3.4.3

./configure  
make  
sudo make install
```

=
###Install pip

We will use pip to install flask.

```
sudo apt-get install python-pip
```

=
###Install Flask

```
sudo pip install Flask
```

=
###Install and Enable Mod_wsgi

Mod_wsgi is an interface between web servers and python web applications. It is an Apache HTTP server mod that enables Apache to serve Flask applications.

**Install**

```
sudo apt-get install libapache2-mod-wsgi python-dev
```

**Enable**

```
sudo a2enmod wsgi
```

=
###Build the Flask Blog Application

We are going to use git to clone the flask blog app which is here: https://github.com/archerydwd/flask_blog

**Install git**

```
sudo apt-get install git
```

First in your home directory, we are going to create a directory called git:

```
mkdir git
cd git/
```

Now issue the following command to download the git repo:

```
git clone https://github.com/archerydwd/flask_blog.git
cd flask_blog
```

We are going to make a few changes to this application:

In the /git/flask_blog/ directory, do:

```
mkdir flask_blog
mv * flask_blog/
touch flask_blog.wsgi
cd flask_blog/
mkdir static
vim __init__.py
```

When editing the above __init__.py file, make the following change:
* Change line 21 from connection = sqlite3.connect('blogdata.db', check_some_thread=False) to connection = sqlite3.connect('/home/darren/git/flask_blog/flask_blog/blogdata.db', check_some_thread=False) making sure to replace darren with your own name shown on your terminal, eg: darren@darren-latitude-E6420:/$

Next we need to change the permissions on the blogdata.db file and on its containing folder for sqlite to be readable and written to.

```
cd git/flask_blog/
sudo chmod 777 flask_blog
cd flask_blog/
sudo chmod 777 blogdata.db
```

=
###Configure and Enable a New Virtual Host

**Create and edit the config file for the site**

```
sudo touch etc/apache2/sites-available/flask_blog.conf
sudo vim etc/apache2/sites-available/flask_blog.conf
```

Insert the following:

```
<VirtualHost *:80>
        ServerName 127.0.0.1:80
        WSGIScriptAlias / /home/darren/git/flask_blog/flask_blog.wsgi
        <Directory /home/darren/git/flask_blog/flask_blog/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /home/darren/git/flask_blog/flask_blog/static
        <Directory /home/darren/git/flask_blog/flask_blog/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

**Disable other virtual hosts**

If you are unsure what other virtual hosts are running, you can issue  the following command:

```
sudo ls etc/apache2/sites-enabled/
```

The above will produce a list of all running hosts, use the names in the below command one after another.

```
sudo a2dissite ror_sakila.conf
```

**Enable the virtual host for flask_blog**

```
sudo a2ensite flask_blog.conf
```

**Edit the wsgi file**

Change directory to git/flask_blog/ and do the following:

```
vim flask_blog.wsgi
```

And insert the following:

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/home/darren/git/flask_blog/")

from flask_blog import app as application
application.secret_key = 'ThisIsMyVerySecretKey'
```

**Restart Apache**

To make our changes take effect.

```
sudo etc/init.d/apache2 reload
```

That should be it for the flask_blog application. You should now be able to open a browser and go to http://localhost and view the blog application running through apache.

=
###Build the Flask Sakila Application

**Install and set up mysql**

When it asks for a password, you can set it to what you want. For the purposes of this I will use the password "password"

```
sudo apt-get install mysql-server
```

Start the MySQL Server

```
sudo etc/init.d/apache2 start
```

**Install msql-connector-python**

```
sudo pip install mysql-connector-python --allow-external mysql-connector-python
```

**Create the database**

To create the database, we need to login and enter a few commands. Please note, if this is your first time using mysql, the first time you login and enter a password, this acts as setting a password. If you don't want to set a password (bad idea) just hit enter when it requests the password.

```
mysql -u root -p
create database flask_sakila;
use flask_sakila;

source /home/darren/git/flask_sakila/flask_sakila/flask_sakila_dump.sql
```

Then to check that this has indeed worked, you can enter the following command and you should see a list of the tables in the database:

```
show tables;
```

**Create the sakila app**

Change directory to the git folder and then:

```
git clone https://github.com/archerydwd/flask_sakila.git
cd flask_sakila/
mkdir flask_sakila/
mv * flask_sakila/
touch flask_sakila.wsgi
cd flask_sakila/
mv main.py __init__.py
mkdir static
vim __init__.py
```

Then while editing __init__.py, make the following changes:
* At the bottom of the file you will see the line: app.config['DB_PASSWORD'] = '' add the password you used when setting up mysql here, for me it will be: app.config['DB_PASSWORD'] = 'password'
* Also at the end of the file you will see a line: if __name__ == "__main__": This line needs to be moved below the app.config['DB'] = 'flask_sakila' line and above the app.run() line. This is because apache will not run anything inside the if statement: if __name__ == "__main__": so we need to move all the app.config lines out of it.

=
###Configure and Enable a New Virtual Host

**Create and edit the config file for the site**

```
sudo touch etc/apache2/sites-available/flask_sakila.conf
sudo vim etc/apache2/sites-available/flask_sakila.conf
```

Insert the following:

```
<VirtualHost *:80>
        ServerName 127.0.0.1:80
        WSGIScriptAlias / /home/darren/git/flask_sakila/flask_sakila.wsgi
        <Directory /home/darren/git/flask_sakila/flask_sakila/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /home/darren/git/flask_sakila/flask_sakila/static
        <Directory /home/darren/git/flask_sakila/flask_sakila/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

**Disable other virtual hosts**

If you are unsure what other virtual hosts are running, you can issue  the following command:

```
sudo ls etc/apache2/sites-enabled/
```

The above will produce a list of all running hosts, use the names in the below command one after another.

```
sudo a2dissite flask_blog.conf
```

**Enable the virtual host for flask_sakila**

```
sudo a2ensite flask_sakila.conf
```

**Edit the wsgi file**

Change directory to git/flask_sakila/ and do the following:

```
vim flask_sakila.wsgi
```

And insert the following:

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/home/darren/git/flask_sakila/")

from flask_sakila import app as application
application.secret_key = 'ThisIsMyVerySecretKey'
```

**Restart Apache**

To make our changes take effect.

```
sudo etc/init.d/apache2 reload
```

That should be it for the flask_sakila application. You should now be able to open a browser and go to http://localhost and view the sakila application running through apache.


