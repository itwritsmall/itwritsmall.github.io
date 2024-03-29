---
title: Installing Nagios with Apache on Raspbian or Ubuntu
date: 2019-06-06 00:00:00 Z
categories:
- blog
tags:
- nagios
- raspbian
- ubuntu
- apache
- monitor
- alert
layout: blog
summary: How to install Nagios with Apache on Raspbian or Ubuntu to monitor your small/home
  network.
comments: false
---

## Introduction

Nagios is one legacy program among many, many more polished open source monitoring programs.  In its free form, it's not what anyone would choose in a large, complex environment.  Still, for single site, small-ish networks (say, less than 100 devices) it's easy to set up, doesn't require a lot of resources, and is extremely flexible.

## Goals

In this tutorial, you will see how to:

* Install Nagios from source
* Configure the Apache server (and PHP) to serve the Nagios web interface
* Edit the Nagios configuration files to start simple up/down monitoring of your network

## Prerequisites

Part of the purpose of this tutorial is to show that you can do something useful on your home/small network using a Raspberry Pi.  If you follow along with this you will accomplish that goal.  I've also checked the tutorial against an Ubuntu Server installation using very minimal resources.  If you want a more production ready installation you can allocate real or virtual resources to that.

### Raspberry Pi

I'm running this installation on a Model B Plus Rev 1.2 Raspberry Pi with a 64GB SD card.  I'm guessing a little less would work and a little more would get the job done too.

### or Virtual Resources

I've also tested this on fairly minimal virtual resources.  Specifically a Hyper-V instance with a single CPU, 1GB of memory and 40GB of hard drive space.  The virtualization platform you choose should matter little and more resources wouldn't hurt.

### Raspbian Installation

I'm running the 2018-11-13 Stretch Lite version Of Raspbian.  The remainder of the configuration options aren't important (other than getting it onto your network) but you'll probably be well served by configuring an SSH connection to do the set up and future management.

### or Ubuntu Installation

I've also tested this with an installation of Ubuntu Server 18.04.2 running on Hyper-V.  Just a basic installation, no additional software only OpenSSH.

### Either OS

Make sure you've upgraded your installation before starting by doing the following:

```bash
sudo apt-get update
sudo apt-get upgrade
```

Let's get started then.

## Tutorial

We'll do a simple install of Nagios to get started doing some simple monitoring.  Simply:

1. Install Pre-Requisite Packages
1. Create Users and Groups
1. Install Nagios Core and Web Interface
1. Install Nagios Plug-ins
1. Set Up Nagios as a Service
1. Configure Simple Monitoring

### Step 1- Installing Pre-Requisite Packages

To display the Nagios Web Interface you will need to install Apache and PHP.

``` bash
sudo apt install apache2 libapache2-mod-php7.0 php7.0
```

> **For Ubuntu** use `sudo apt-get install apache2 libapache2-mod-php php7.0 build-essential unzip` instead.

> **Note:** If you want to see if these packages are installed already use the `apt list --installed` command.

After installing these packages, configure `php`.

Set the `php` timezone:

``` bash
sudo nano /etc/php/7.0/apache2/php.ini
```

> **For Ubuntu** use `sudo nano /etc/php/7.2/apache2/php.ini`

Change;

``` bash
;date.timezone =
```

to

``` bash
Date.timezone = <appropriate variable>
```

Like such:

![Change PHP date setting]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/setPHPDate.png)

> **Note:** use the list here to find your appropriate variable: <http://php.net/manual/en/timezones.america.php>

Save your changes.

Create a `php.info` file:

``` bash
sudo echo '<?php phpinfo(): ?>' | tee /var/www/html/info.php
```

> **For Ubuntu** `sudo echo '<?php phpinfo(): ?>' | sudo tee /var/www/html/info.php`

Now, enable a couple of Apache Modules

``` bash
sudo a2enmod rewrite
sudo a2enmod cgi
```

Both commands will inform you that the `apache2` service needs to be restarted.

![Change PHP date setting]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/enableApacheModules.png)

Let's follow that advice and restart `apache2`:

``` bash
sudo systemctl restart apache2
```

Test that Apache is running:

``` bash
sudo netstat -tlpn
```

You should see something similar to this:

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/testApacheNetstat.png)

Which shows that `apache2` is listening for web requests at port 80.

You can also go to `http://<your machine name or IP>` and see something like this:

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/testApacheBrowser.png)

Which shows that `apache2` "works!".

Test your `php` configuration.  Open a web browser and navigate to `http:<machine ip>/info.php` and scroll down to the **Default timezone** setting and confirm.

You should see the time zone you configured like:

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/testPhpDate.png)

Cool, you're serving up PHP pages on Apache.  Let's get into the Nagios stuff.

### Step 2- Create Users and Groups

The Nagios program needs a user to run as.  Create the user by:

``` bash
sudo useradd -m nagios
```

Then set a password for the user:

``` bash
sudo passwd nagios
```

You will set and confirm the password like this:

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/setNagiosUserPassword.png)

Now, set up a group `nagcmd` that you can give some rights to:

``` bash
sudo groupadd nagcmd
```

Now add the `nagios` user you created to the group so that it gets those rights:

``` bash
sudo usermod -a -G nagcmd nagios
```

Also add the `www-data` user to that group.  

``` bash
sudo usermod -a -G nagcmd www-data
```

You've got a `nagios` user account waiting to do some work.  Let's install Nagios to give her a place to do that.

### Step 3- Install Nagios Core and Web Interface

We're going to download the latest version of Nagios Core and install it from source.

Change into the home directory and get the latest version (4.4.3 as of this writing) of Nagios Core:

``` bash
cd ~
sudo wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.3.tar.gz
```

> **Note:** Check that url in a web browser...from time to time it changes.

You'll see the progress of the download:

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/getnagiosCoreSource.png)

Once it is all downloaded, unpack the files:

``` bash
sudo tar zxvf nagios-4.4.3.tar.gz
```

Let's start building Nagios.  First configure the source code:

``` bash
cd nagios-4.4.3
sudo ./configure --with-command-group=nagcmd
```

Using Raspbian (less so with Ubuntu/Hyper-V), this will take some time, and you'll see a lot of this:

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/configureNagiosCoreProgress.png)

Wrapping up with something like this:

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/configureNagiosCoreComplete.png)

Now we've got configured the Nagios Core source.  Let's get to the compiling:

``` bash
sudo make all
```

Now if you're doing this on the Pi, you're going to have some time while it cranks through the work like this:

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/makeNagiosCoreProgress.png)

The `make` command you issued is creating the Nagios Core executables, libraries, directories, etc.  When the process wraps, you'll see something like this:

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/makeNagiosCoreComplete.png)

So enjoy, but not for too long because there's a lot of installing left to be done.

These steps will install and configure the binaries, documentation, web interface, external command directory, init script, sample configuration files and the web interface.

Get with the make-ing.

``` bash
sudo make install
sudo make install-commandmode
sudo make install-init
sudo make install-config
sudo make install-webconf
```

Now, install the configuration file to the correct Apache directory.

``` bash
sudo install -c -m 644 sample-config/httpd.conf /etc/apache2/sites-enabled/nagios.conf
```

Finally, let's set a password for the Nagios Admin (`nagiosadmin`) user that you will use to log into the web interface.

```bash
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```

This command will present you with a familiar set/confirm password process.

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/setNagiosAdminPassword.png)

Let's bounce Apache.

``` bash
sudo systemctl restart apache2
```

Start up Nagios
First, let's make sure everything is cool with the Nagios configuration.

``` bash
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

Then, let's start up the Nagios program.

```bash
 sudo /usr/local/nagios/bin/nagios -d /usr/local/nagios/etc/nagios.cfg
```

We will make that easier in a couple of steps.  Meantime, test your work by pointing a web browser at `http://<machine name or IP>/nagios`.  You'll get an authentication prompt:

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/testNagiosWebInterface.png)

After successfully authenticating you will see this:

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/testNagiosWebInterfaceSuccess.png)

We've got everything we need to do simple monitoring.  Let's do a little get-ahead work in the next step and install Nagios Plugins then do a little work to make Nagios start as a service.

### Step 4- Install the Nagios Plugins

There is a set of Plugins maintained that will let you do more than simple up-down monitoring.  While we won't configure anything more than up-down monitors in this tutorial, we will install the plugins for future use.  Let's get the current version of the Nagios Plugins:

``` bash
cd ~
sudo wget https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz
```

Now, extract the compressed files.

``` bash
sudo tar zxvf nagios-plugins-2.2.1.tar.gz
```

Next roll through the Plugin install process.  Configure.  Compile.  Install.

``` bash
cd ~
cd nagios-plugins-2.2.1
sudo ./configure --with-nagios-user=nagios --with-nagios-group=nagcmd
sudo make
sudo make install
```

> **Warning:** On Raspbian, both the `configure` and `make` command will take some time.

That installs your plug-ins.  We won't use them right out of the chute, but they will allow you to do some very complex monitoring and alerting later on.  The next thing we will want to do is set up the Nagios Core as a service.

### Step 5- Set Up Nagios as a Service

You (obviously?) want Nagios to be running more or less continuously.  This is a simple process.  Create a dynamic link from `nagios` in the `init.d` directory to the `rc5.d` directory

``` bash
sudo ln -s /etc/init.d/nagios /etc/rc5.d/nagios
```

Now, let's start up Nagios as a service for the first time.

 ``` bash
sudo systemctl start nagios
sudo systemctl enable nagios
```

You can test it at any time.
``` bash
sudo systemctl staus nagios
```

If you go back to the web interface (`https://<machine name or IP>/nagios`) and look at the menu on the left side you will see a **Current Status** section.

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/nagiosWebInterfaceMenu.png)

Click on the **Hosts** selection.

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/nagiosWebInterfaceLocalhostStatus.png)

This is the status of the machine/instance that Nagios is running on.  Interesting, but not super useful.  In the next step we'll make some modifications to Nagios config files so that you're monitoring more hosts on your network.

### Step 6- Configure Simple Monitoring

Seems like a good time to make Nagios do some work for you after all you've done to bring it to life.  Let's configure Nagios to monitor some of your hosts.  First, change the `nagios.cfg` file to use a file with information about your hosts.

``` bash
cd /usr/local/nagios/etc
sudo nano nagios.cfg
```

Scroll down and find the section titled `#OBJECT CONFIGURATION FILE(S)`, then add a line like `cfg_file=/usr/local/nagios/objects/<descriptive name>.cfg` similar to this:

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/updateNagiosConfigFile.png)

> **Note:** You can add a comment in any of the Nagios config files by preceding the line with a `#`.  Remember, comments win wars.

Now create a host file that's named and located where you told the `nagios.cfg` file it would be.

``` bash
cd /usr/local/nagios/etc/objects
sudo nano <descriptive name you specified>.cfg
```

In the file, create as many host definitions as you like.  Again, this is for really simple monitoring right now so we're going to do the simplest configuration.  The format for the host definitions is:

``` bash
define host {
        use                 linux-server;
        host_name           <yourhost>;
        alias               <Display Name>;
        address             <host IP Address>;
        max_check_attempts  10;
}
```

Here's an example:

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/sampleNagiosHostObject.png)

The `host_name` is the fully qualified domain name of the host.  In the `alias` field you can put a more friendly name.  Here I used the host name and the service that runs on it.  Lastly, the `address` field is the IP address of the host you are monitoring.

> **Note:** Technically, you only need either a `host_name` or a `address` specified.

Save up the file.  Nagios won't use the new configurations until you reload the service.  You can do it like this.  First check the configuration:

``` bash
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

You should see something like this (depending on how many hosts you defined):

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/startNagiosConfigFileCheckAddHosts.png)

Now, restart Nagios:

``` bash
sudo systemctl nagios restart
```

Open up the web interface and go to the **Hosts** selection in the **Current Status** section.  You should see:

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/newHostPending.png)

The **Status** will show **PENDING** until the check runs.  You can see when that will be in the **Status Information** column.  Once that check happens, you should see the status change to something like this:

![image]({{site.url}}/assets/2019-03-09-install-nagios-raspbian/newHostUp.png)

And that's it (for now), you've got up/down monitoring for as many hosts as you want to add.

## Conclusion

End result, you should have simple monitoring for your network.  That's great, but it's simple.  There's a lot more monitoring **and** alerting that Nagios can do for you.  Checking more services than just `ping`, e-mail alerts, grouping hosts into groups, storing historical data is all possible.  I've just got to spend some time setting it up and (of course) writing it down.

### What to do next

* The next logical step is to make Nagios send you e-mail alerts.  Here's the tutorial for that: <https://{{site.url}}/_posts/20019-03-09-install-postfix-nagios.html>

### Resources

* (Official Nagios Core Documentation)[https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/4/en/toc.html]
* <https://www.digitalocean.com/community/tutorialshow-to-install-nagios-4-and-monitor-your-servers-on-centos-7>
* <http://www.d3noob.org/2016/04/installing-nagios-4-on-raspberry-pi.html>
* <https://poweruphosting.com/blog/install-nagios/>

### Revision History

* 03/09/2019- Writing begins
* 06/06/2019- Content review
* 06/06/2019- Style Guide review
* 06/06/2019- Published
* 06/14/2020- Cleaned up some markdown
