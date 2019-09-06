---
layout: post
title: "Installing Netbox with Apache and PostgreSQL on Ubuntu"
categories: [tutorial]
tags: [netbox, apache, postgresql, ubuntu]
excerpt: How-to install Digital Ocean's Netbox on Ubuntu Server
---

## Introduction
This post will walk you through installing Digital Ocean's Netbox DCIM/IPAM application on Ubuntu Server.  The Netbox program is a Django/Python application.  In this tutorial the web application is served up by Gunicorn and Apache.  The application saves data to a PostgreSQL database.  All of this runs on Linux, in this case Ubuntu Server 18.x.

## Contents

## Prerequisites
Ubuntu and Netbox's "stack-mates" PostgreSQL and Apache take up minimal resources.  Use the old server you just retired or carve out a small virtual machine.  For a small network, feel comfortable allocating 1GB of RAM, a single CPU and 30GB of disk space to a virtual machine instance.  Obviously you can give it more resources (within reason) and benefit, but don't hesitate to implement Netbox if you don't have a ton available.

### Installing Ubuntu 18.04 Server
 This document is based on Ubuntu Linux 18.0.4.2 LTS 64-bit Live Server.  Netbox runs on other flavors of Linux, there's nothing special about the choice of Ubuntu and any other Debian variant should work similarly for the purposes of this tutorial.  The installation uses mostly defaults except on the Network Connection screen where you may set up a fixed IP address instead of allowing the server to get an address from DHCP.  If a DHCP reservation works better in your environment, go for it.

### Creating DNS Records
You will need to add a couple of records to your DNS server to support the Netbox installation.
The first will be a Host (A) record for the server.  You will use this FQDN name in the Django configuration later.  More often than not you will also want to use a different friendly name for the application, so the second will be an Alias (CNAME) record that points to the server record you're going to use for the virtual host.  You'll use the alias name later in your Apache virtual host config.

As an example, if your server name is **your_server**, create a host record for that server that resolves to the fixed IP address you set up when you installed Ubuntu.  Then create an alias, such as **netbox.example.com** that points to the **your_server** host record.  

## Step 1- Installing Webmin (Optional)
If you want the option of using a GUI interface to manage this server, install Webmin.

>If your memory and Linux skills are better than mine (the bar is low) you'll be able to get any information that Webmin can provide using the command line.  Me?  I like to have the interface available when can't recall the correct command.  Which is often.  Like all the time. 
 
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
The public key will download and you will see a progress bar like this:

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

## Step 2- Launching Webmin (optional)
Access the Webmin application by opening a browser and going to `https://<your_server_IP>:10000`. And whoa!  Make that go away:

![image]({{site.url}}/assets/2019-08-06-install-netbox/webminSecurityWarning.png)

Once you pass the certificate warning, you will come to a login screen.  

![image]({{site.url}}/assets/2019-08-06-install-netbox/webminLoginScreen.png)

You can use the username you created during the Ubuntu installation.  After clicking **Sign in** you'll get the Webmin dashboard.

![image]({{site.url}}/assets/2019-08-06-install-netbox/webminDashboard.png)

Webmin deserves more explanation, but that's not why you called.  Let's move forward on installing Netbox.  

> This is a good time to take a snapshot if you're installing on a virtual machine.

## Step 3- Installing PostgreSQL
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

At this point, you've got an operating system, the Webmin management tool (if you wanted it) and now a SQL database to store the data for the program.  Next up, creating the databases and user that the Netbox application will need.

## Step 4- Creating the Database
Netbox will need a PostgreSQL database to store data into and a user credential to access that database.  

Change into the PostgreSQL command prompt:

``` console
$ sudo -u postgres psql
```

This will put you into PostgreSQL command mode.  

![image]({{site.url}}/assets/2019-08-06-install-netbox/postgresCommandLine.png)

> Warning: Don't forget the "`;`" at the end of each line.

To set up the database, user and privileges:

``` console
postgres=# CREATE DATABASE <your database name>;
postgres=# CREATE USER <your database user> WITH PASSWORD '<your password>';
postgres=# GRANT ALL PRIVILEGES ON DATABASE <your database name> TO <your database user>;
```
![image]({{site.url}}/assets/2019-08-06-install-netbox/postgresDatabaseSetup.png)

To exit the PostgreSQL command line and return to the Ubuntu command line:
`postgres=# \q`

Test PostgreSQL again, by logging into the database you created with the user you created:

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

## Step 5- Install Netbox and Python
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

Netbox has a long list of Python modules it requires.  Luckily the good people at Digital Ocean compiled a text file of those modules.  Use pip to install this list:

``` console
$ sudo -H pip3 install -r requirements.txt
```
![image]({{site.url}}/assets/2019-08-06-install-netbox/netboxPipInstall.png)

> This process will take a little time.  Many characters and progress bars will ensue.

With Python and the Netbox program installed on the host, you can now do the initial configuration of the program.

## Step 6- Configuring the Netbox Application
The Netbox web application is written in Python using the Django framework.  We're going to do a little configuration to these elements.

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

> Elephant in the room:  You're right, that's a hell of a lot of netbox directories.

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

> NOTE: `ALLOWED_HOSTS` refers to the host that can access the web application.  That is the machine running the reverse proxy, not the machine making the web request.  We're setting all of this up on a single box, so it will be the name of the server you've been doing all this work on.  `ALLOWED_HOSTS` needs to be identical to the name you use in the URL to access this system (less any port number), so if you want to use a FQDN like **http://netbox.example.com** use `ALLOWED_HOSTS = ['netbox.mycorp.com']`.  If you want to use **http://netbox** then use `ALLOWED_HOSTS = ['netbox']`.  To use both, `ALLOWED_HOSTS = ['netbox.mycorp.com','netbox']`.  If you have problems accessing the site later on in this tutorial, then use `ALLOWED_HOSTS =[*]` and allow all hosts access for troubleshooting.

Modify:

``` console
DATABASE = {
	'NAME': '<use the database name you created>',
	'USER':'<the postgresql user you created>'
	'PASSWORD':'<the postgresql password you specified>'
}

And then change:

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

## Step 7- Initiating the Netbox Application
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

## Step 8- Testing the Netbox Application

The `manage.py` script allows you to check out your installation before having Apache and the WSGI server installed and configured.

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

## Step 9- Installing Apache and Configuring Virtual Site
The Netbox application utilizes a reverse proxy (Apache) and a WSGI server (Gunicorn).  The combination of Gunicorn and Apache works like this:  Apache responds to web clients, managing connections and serving up static files.  Apache then reverse-proxies requests to the Gunicorn server running on port 8001 (which you will specify in your Apache Virtual Host config) to serve up the Django code.

Install the Apache Web Server and related libraries:

```console
$ sudo apt-get install apache2 libapache2-mod-wsgi-py3
```

Configure a virtual site for the Netbox web application by creating a site description file:

``` console
$ sudo nano /etc/apache2/sites-available/netbox.conf
```

Put the following information into this file, make sure to change the `<your_server_alias>` parameter to match the Alias (CNAME) record you created a DNS record for:

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

## Step 10- Set Access Rights for Media Directory
Netbox allows you to save image files attached to different objects, but the account that the Netbox application runs as (`www-data`) needs to have ownership of the directory the media is installed in.

Wrap up the Apache configuration by giving the Apache account ownership:

``` console
$ sudo chown -R www-data:www-data /opt/netbox/netbox/media/
```

## Step 11- Installing and Configuring WSGI
I had not installed a Django application before, so configuring a WSGI server was new to me.  The Netbox doumentation suggests using Gunicorn ("Green Unicorn"), a Python WSGI server.  

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

## Step 12- Installing and Configuring Supervisor
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

## Step 13- Launching Netbox
Moment of truth, you should be able to reach the home page of the Netbox web application by using a browser to navigate to `http://<your_server_alias>`

## Conclusion
Take one final snapshot of your virtual machine if that's where you're installing it.  Next, start using the program.  Next up for me is putting together a series on using the program to document your infrastructure.

### Sources
* Digital Ocean Netbox Github Page https://github.com/digitalocean/netbox
* Netbox Documentation https://netbox.readthedocs.io/en/stable/

### Revision History
* 11/8/2018-GM Initial post.  Enjoy.


