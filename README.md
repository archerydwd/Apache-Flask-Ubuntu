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
vim __init__.py
```

When editing the above __init__.py file, make the following changes:
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





















