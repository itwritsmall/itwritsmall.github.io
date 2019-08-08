---
layout: post
title: "Installing Netbox with Apache and PostgreSQL on Ubuntu 18.x"
categories: [tutorial]
tags: [netbox, apache, postgresql, ubuntu]
excerpt: How-to install Digital Ocean's Netbox on Ubuntu Server
---

## Introduction
This post will walk you through installing Digital Ocean's Netbox DCIM/IPAM application on Ubuntu Server.  The Netbox program is a Django/Python application.  In this tutorial the web application is served up by Gunicorn and Apache.  The application saves data to a PostgreSQL database.  All of this runs on Linux, in this case Ubuntu Server.

## Prerequisites
### Host Requirements
Ubuntu and Netbox's "stack-mates" PostgreSQL and Apache take up minimal resources.  Use the old server you just retired or carve out a small virtual machine.  For a small network, feel comfortable allocating 1GB of RAM, a single CPU and 30GB of disk space to a virtual machine instance.  Obviously you can give it more resources (within reason) and benefit, but don't hesitate to implement Netbox if you don't have a ton available.

### Install Ubuntu 18.04 Server
 This document is based on Ubuntu Linux from the 18.0.4.2 LTS 64-bit Live Server.  Netbox runs on other flavors of Linux, there's nothing special about the choice of Ubuntu.  The installation uses mostly defaults except on the Networks Connections screen where you may set up a fixed IP address instead of allowing the server to get an address from DHCP.  If a DHCP reservation works better in your environment, go for it.

### Create DNS Records
You will need to add a couple of records to your DNS server to support the Netbox installation.
The first will be a Host record for the server.  You will use this FQDN name in the Django configuration later.  More often than not you should use a different friendly name for the application, so the second will be an alias to the server name you're going to use for the virtual host.  You'll use the alias name later in your Apache virtual host config.

As an example, if your server name is your_server, create a host record for that server that resolves to the fixed IP address you set up when you installed Ubuntu.  Then create an alias, such as netbox.example.com that points to the your_server host record.  

## Step 1- Updating the server
 When I installed Ubuntu 18.04.1 I wasn't able to install some Python packages.  By adding the "universe" repositories I was able to load the community-maintained open-source software packages.  On another install (which may have been using an earlier 18.xx release) the "universe" was included.

Open the repository source file:

``` bash
$ sudo nano /etc/apt/sources.list
```

Add universe at the end of each line, like this:
``` config
deb http://archive.ubuntu.com/ubuntu bionic main universe
deb http://archive.ubuntu.com/ubuntu bionic-security main universe 
deb http://archive.ubuntu.com/ubuntu bionic-updates main universe
```

Save and quit the text editor then update and upgrade your base install.

Now, update the packages list and upgrade any that are not current.

``` bash
$ sudo apt-get update
$ sudo apt-get upgrade
```

You now have a clean, updated server.  Next, you can install the Webmin graphic management tool if you want.

## Step 2- Installing Webmin (Optional)
If you want the option of using a GUI interface to manage this server, install Webmin.

>If your Linux skills are better than mine (the bar is low) you'll be able to get any information that Webmin can provide using the command line.  Me?  I like to have the interface available if I can't recall the correct command.  Which is often.  Like all the time. 
 
Add the webmin repository to your sources list:

``` console
$ sudo nano /etc/apt/sources.list
```

Add the following line:

``` console
deb http://download.webmin.com/download/repository sarge contrib
```

Save and close the text editor. 

Get and add the webmin public key:

``` console
$ wget http://www.webmin.com/jcameron-key.asc
```
The public key will download and yo will see a progress bar like this

![image]({{site.url}}/assets/2019-08-06-install-netbox/wgetPublicKey.png)

``` console
$ sudo apt-key add jcameron-key.asc
```

Update your package lists, then install the Webmin package:

``` console
$ sudo apt-get update
$ sudo apt-get install webmin
```
Sit back and watch as the prerequisites and the program are installed.  That's all there is to installing Webmin on the host.  You can now move on to launching the program.

## Step 3- Launching Webmin (optional)
Access the Webmin application by opening a browser and going to `https://<your_server_IP>:10000`. And whoa!

![image]({{site.url}}/assets/2019-08-06-install-netbox/webminSecurityWarning.png)

Once you pass the certificate warning, you will come to a login screen.  

![image]({{site.url}}/assets/2019-08-06-install-netbox/webminLoginScreen.png)

You can use the username you created during the Ubuntu installation.  After clicking Sign in you'll get the Webmin dashboard.

![image]({{site.url}}/assets/2019-08-06-install-netbox/webminDashboard.png)

This tool deserves more explanation, but just not right now.  Let's move forward on installing Netbox.  

> This is a good time to take a snapshot if you're installing on a virtual machine.

## Step 4- Installing PostgreSQL
The Netbox web application needs a database to write to, this tutorial uses a PostgreSQL database.  

Install PostgreSQL and related libraries:

``` console
$ sudo apt-get install postgresql libpq-dev
```

The PostgreSQL install returns a lot of info:

![image]({{site.url}}/assets/2019-08-06-install-netbox/postgresInstall.png)

> Note: By default the install is putting data into the /var/lib/postgresql/10/main directory and logs into the /var/log/postgresql/postgresql-10-main.log file.

You can quickly test the PostgreSQL installation:

```console
$ sudo systemctl status postgresql
```

![image]({{site.url}}/assets/2019-08-06-install-netbox/sysctlPostgresStatus.png)

This should produce something similar to the above.  Use `CTRL + c` to return to the Ubuntu command prompt.

At this point, you've got an operating system, a management tool (if you wanted it) and now a SQL database to store the data for the program.  Next up, creating the database and user the program will need.

## Step 5- Creating the Database
Netbox will need a PostgreSQL database to store data into and a user credential to access that database.  

Change into the PostgreSQL command prompt:

``` console
$ sudo -u postgres psql
```

This will put you into PostgreSQL command mode.  

![image]({{site.url}}/assets/2019-08-06-install-netbox/postgresComandLine.png)

> Warning: Don't forget the "`;`" at the end of each line

To set up the database, user and privileges:

``` console
postgres=# CREATE DATABASE database_name;
postgres=# CREATE USER database_user WITH PASSWORD 'your_password';
postgres=# GRANT ALL PRIVILEGES ON DATABASE database_name TO database_user;
```
![image]({{site.url}}/assets/2019-08-06-install-netbox/postgresDatabaseSetup.png)

To exit the PostgreSQL command line and return to the Ubuntu command line:
`postgres=# \q`

Test PostgreSQL again, logging into the database you created with the user you created:

```console
$ psql -U database_user -W -h localhost database_name
```
![image]({{site.url}}/assets/2019-08-06-install-netbox/postgresTestDatabase.png)

Which should get you something similar to the above.  To exit to the Ubuntu command prompt:

``` console
database_name=# \q
```

You've got a database for the program to use, time to move on to installing the Netbox program.

> Great time to take another snapshot before you move on to installing Netbox.

## Step 6- Install Netbox and Python
Now it's time to install the Netbox web application and all of the Python components it requires.  First, install the required Python modules and other libraries:

``` console
$ sudo apt-get install python3-dev python3-pip build-essential libxml2-dev libxslt1-dev libffi-dev libssl-dev
```

> This is a long list of packages, be patient.

Next, create a directory for the Netbox program and move into that folder:

``` console
$ sudo mkdir -p /opt/netbox
$ cd /opt/netbox
```

Use the git program to download the current version of Netbox.

> Make sure you include the  `SPACE + .` at the end of the line or this whole thing will burst into flames.

``` console
$ sudo git clone -b master https://github.com/digitalocean/netbox.git .
```

![image]({{site.url}}/assets/2019-08-06-install-netbox/gitCloneNetbox.png)

Netbox has a long list of Python modules it requires.  Luckily Digital Ocean compiled a text file of those modules.  Use pip to install this list:

``` console
$ sudo -H pip3 install -r requirements.txt
```
![image]({{site.url}}/assets/2019-08-06-install-netbox/netboxPipInstall.png)

> This process will take a little time.  Many characters and progress bars will ensue.

With Python and the Netbox program installed on the host, you can now do the initial configuration of the program.

## Step 7- Configuring the Netbox Application
The Netbox web application is written in Python using the Django framework.  Do a little configuration to these elements.

The Netbox application uses a secret key, which you can create with an included Python script.  

Change into the correct directory and run the script:

``` console
$ cd /opt/netbox/netbox
$ sudo ./generate_secret_key.py
```
![image]({{site.url}}/assets/2019-08-06-install-netbox/netboxGenerateSecretKey.png)

Copy and paste the output somewhere (or show off by memorizing it), you'll need to place it in the configuration file.  

Change into the correct directory and copy the example config file into a working config file:


``` console
$ cd /opt/netbox/netbox/netbox
$ sudo cp configuration.example.py configuration.py
```

> Note:  OK, so that's a hell of a lot of netbox directories.

Now edit the configuration file:

``` console
$ sudo nano /opt/netbox/netbox/netbox/configuration.py
```

Make the following changes:

``` console
ALLOWED_HOSTS = []
```

To

``` console
ALLOWED_HOSTS = ['put the FQDN of your server here']
```

> NOTE: `ALLOWED_HOSTS` refers to the host that can access the web application.  That is the machine running the reverse proxy, not the machine making the web request.  We're setting all of this up on a single box, so it will be the name of the server you've been doing all this work on.  `ALLOWED_HOSTS` needs to be identical to the name you use in the URL to access this system (less any port number), so if you want to use a FQDN like http://netbox.mycorp.com use `ALLOWED_HOSTS = ['netbox.mycorp.com']`.  If you want to use http://netbox then use `ALLOWED_HOSTS = ['netbox']`.  To use both, `ALLOWED_HOSTS = ['netbox.mycorp.com','netbox']`.  If you have problems accessing the site later on in this tutorial, then use `ALLOWED_HOSTS =[*]` and allow all hosts access for troubleshooting.

``` console
DATABASE = {
	'NAME': '<use the database name you created>',
	'USER':'<the postgresql user you created>'
	'PASSWORD':'<the postgresql password you specified>'
}

SECRET_KEY = ''
```

To

``` console 
SECRET_KEY= '<the key you generated with generate_secret_key.py>'
```

Your changes will look similar to this:

![image]({{site.url}}/assets/2019-08-06-install-netbox/netboxConfigurationFile.png)

When you've made the changes, save the file and exit the text editor. 

Now that the Django configuration is in order it's time to run some commands to do the initial setup for the Netbox program. 

## Step 8- Initiating the Netbox Application
The Netbox program includes a Python script to initialize the program.

Run the provided manage.py Python script to set up the program.  It's in the program directory:

``` console
$ cd /opt/netbox/netbox/
$ sudo python3 manage.py migrate
```
From which will spew much progress:
![image]({{site.url}}/assets/2019-08-06-install-netbox/netboxMigration.png)

Create an admin user:

``` console
$ sudo python3 manage.py createsuperuser
```
![image]({{site.url}}/assets/2019-08-06-install-netbox/netboxCreateSuperuser.png)

Obviously, substitute a valid e-mail address of your own.  I left the Username as root but there is a good argument to be made for changing it.

Create the static HTML pages the program uses.

``` console
$ sudo python3 manage.py collectstatic --no-input
```
![image]({{site.url}}/assets/2019-08-06-install-netbox/netboxCreateStaticHTML.png)

Optionally (I recommend it) load a few initial data objects you can build your inventories from.

``` console
$ sudo python3 manage.py loaddata initial_data
```
![image]({{site.url}}/assets/2019-08-06-install-netbox/netboxInstallData.png)

At this point the Netbox application is ready for testing.

## Step 9- Testing the Netbox Application

The `manage.py` script allows you to check out your installation without having Apache and the WSGI server installed.

Use the `manage.py` script to test your installation:  

``` console
$ cd /opt/netbox/netbox
$ sudo python3 manage.py runserver 0.0.0.0:8000 --insecure
```
![image]({{site.url}}/assets/2019-08-06-install-netbox/netboxTestServer.png)

In a web browser navigate to `http://<your_server_FQDN>:8000`

![image]({{site.url}}/assets/2019-08-06-install-netbox/netboxFirstLaunch.png)

You should see your new best DCIM/IPAM friend.  Take a look around the exit by using `CTRL + C` in the console. 

You don't want to use the `manage.py` script in production so we will move on to installing Apache as a reverse proxy.

## Step 10- Installing Apache and Configuring Virtual Site
The Netbox application utilizes a reverse proxy (Apache) and a WSGI server (Gunicorn).  The combination of Gunicorn and Apache works like this:  Apache responds to web clients, managing connections and serving up static files.  Apache then reverse-proxies requests to the Gunicorn server running on port 8001 (which you will specify in your Apache Virtual Host config).

Install the Apache Web Server and related libraries:

```console
$ sudo apt-get install apache2 libapache2-mod-wsgi-py3
```

Configure a virtual site for the Netbox web application by creating a site description file:

``` console
$ sudo nano /etc/apache2/sites-available/netbox.conf
```

Put the following information into this file, make sure to change the `<your_server_alias>` parameter to match the alias you created a DNS record for:

``` console
<VirtualHost *:80>
    ProxyPreserveHost On

    ServerName <your_server_alias>

    Alias /static /opt/netbox/netbox/static

    # Needed to allow token-based API authentication
    WSGIPassAuthorization on

    <Directory /opt/netbox/netbox/static>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride None
        Require all granted
    </Directory>

    <Location /static>
        ProxyPass !
    </Location>

    RequestHeader set "X-Forwarded-Proto" expr=%{REQUEST_SCHEME}
    ProxyPass / http://127.0.0.1:8001/
    ProxyPassReverse / http://127.0.0.1:8001/
</VirtualHost>
```

Save things up then close your text editor.

Enable some supporting Apache modules:

``` console
$ sudo a2enmod proxy
$ sudo a2enmod proxy_http
$ sudo a2enmod headers
```
Each of these commands will notify you that you have to restart Apache.

![image]({{site.url}}/assets/2019-08-06-install-netbox/apacheA2enmod.png)

We will take care of that in a bit.

Then enable the Netbox virtual site:

``` console
$ sudo a2ensite netbox
```

Finally, reload the Apache service so all of your enabling gets enabled:

``` console
$ sudo systemctl reload apache2
```

Apache is set up, but since it's a reverse proxy there's not much to see until you set up the WSGI server.  Before we get to that though, allow Apache access to a media directory.

## Step 11- Set Access Rights for Media Directory
Netbox allows you to save image files attached to different objects, but the account that the Netbox application runs as (`www-data`) needs to have ownership of the directory the media is installed in.

Wrap up the Apache configuration by giving the Apache account ownership:

``` console
$ sudo chown -R www-data:www-data /opt/netbox/netbox/media/
```

## Step 12- Installing and Configuring WSGI
I had not installed a Django application before, so configuring a WSGI server was new to me.  The Netbox doumentation suggests Gunicorn ("Green Unicorn") a python WSGI server.  

Gunicorn has the capability of serving up the static pages included in the Netbox application, so you could conceivably eliminate the moving part that is Apache but given the low resource requirements there is probably not a lot of win there.  If your installation gets heavy use, you may even see some performance gains by having that layer.

Because gunicorn is a Python program, the Python installer program PIP is used: 

``` console
$ sudo pip3 install gunicorn
```
![image]({{site.url}}/assets/2019-08-06-install-netbox/gunicornInstall.png)

Create a configuration file for Gunicorn:

``` console
$ sudo nano /opt/netbox/gunicorn_config.py
```

Place the following into the file, note that the binding (`localhost:8001`) matches the port specified in the Apache Virtual Host configuration.

``` console
command = '/usr/bin/gunicorn'
pythonpath = '/opt/netbox/netbox'
bind = '127.0.0.1:8001'
workers = 3
user = 'www-data'
```

Save and exit the text editor.

That's it for the WSGI server.  It doesn't run as a Linux service though, so install and configure Supervisor to take care of keeping it running.

## Step 13- Installing and Configuring Supervisor
Gunicorn doesn't start itself up automatically when Apache does, so you'll need to install and configure the program Supervisor to make sure that happens.

Install the Supervisor Package:

``` console
$ sudo apt-get install supervisor
```

Create a configuration file for the gunicorn program running the netbox WSGI:

``` console
$ sudo nano /etc/supervisor/conf.d/netbox.conf
```

Configure it as follows:

``` console
[program:netbox]
command = gunicorn -c /opt/netbox/gunicorn_config.py netbox.wsgi
directory = /opt/netbox/netbox/
user = www-data

[program:netbox-rqworker]
command = python3 /opt/netbox/netbox/manage.py rqworker
directory = /opt/netbox/netbox/
user = www-data
```

Save and close the text editor, then restart the Supervisor service:

``` console
$ sudo service supervisor restart
```

With the WSGI server and Reverse Proxy running, you're ready to actually use the Netbox application.

## Step 14- Launching Netbox
Moment of truth, you should be able to reach the home page of the Netbox web application by using a browser to navigate to `http://<your_server_alias>`

## Conclusion
Take one final snapshot of your virtual machine if that's where you're installing it.  Next, start using the program.  Next up for me is putting together a series on using the program to document your infrastructure.

### Sources
* Digital Ocean Netbox Github Page https://github.com/digitalocean/netbox
* Netbox Documentation https://netbox.readthedocs.io/en/stable/

### Revision History
* 11/8/2018-GM Initial post.  Enjoy.


