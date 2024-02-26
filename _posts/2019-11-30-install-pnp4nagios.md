---
title: Installing PNP4Nagios and RRDTool for Nagios on Debian Linux
date: 2019-11-30 00:00:00 Z
categories:
- blog
tags:
- Nagios
- RRDTool
- PNP4Nagios
- Debian
layout: blog
summary: How to install PNP4Nagios and RRDTool for Nagios on Debian Linux.
comments: false
---

## Introduction

By default, Nagios does not process and store the performance data it receives from checks but it does allow you to pass the data to an external program.  Installing PNP4Nagios allows you to store the data in RRD (Round Robin Database) Databases, and provides a rudimentary GUI to display the data.  Installing RRDTool will provide the ability to host and manage the RRD databases.

## Goals

This tutorial will guide you through:

* Installing RRD Tool and other PNP4Nagios
 prerequisite packages
* Downloading and Installing PNP4Nagios
 from source
* Configure Nagios to pass data to the PNP4Nagios
 Program
* Test that PNP4Nagios
 is passing data to the RRD databases
* Configure Nagios to display PNP4Nagios
 stored performance data

## Prerequisites

The tutorial assumes that you have installed Nagios on Linux already.  My tutorial for installing Nagios onto Debian Linux is [here](https://{{site.url}}/_posts/2019-06-06-install-nagios-raspbian.md).

## Tutorial Steps

1. Installing RRDTools
1. Downloading PNP4Nagios
1. Compiling and Installing PNP4Nagios
1. Configuring NPCD service
1. Restarting Apache
1. Modifying the `nagios.cfg` file
1. Defining Nagios Commands
1. Verifying the Nagios Configuration
1. Verifying PNP4Nagios
1. Removing `install.php` file
1. Modifying the `templates.cfg` file
1. Modifying the Generic Host and Service definitions
1. Testing the new Nagios configuration and restarting
1. Viewing the performance graphs integrated into the Nagios GUI

### Step 1- Installing RRDTool

RRDTool is maintained as a Linux package, so install that and some other prerequisites using `apt-get`.

``` BASH
sudo apt-get install -y rrdtool librrds-perl php-gd php-xml
```

Next, we'll grab the PNP4Nagios code.

### Step 2- Downloading PNP4Nagios Source

PNP4Nagios isn't maintained as a package, so we'll download it from the project's GitHub site.

``` BASH
cd ~
sudo wget -O PNP4Nagios.tar.gz https://github.com/lingej/PNP4Nagios/archive/0.6.26.tar.gz
sudo tar xzf PNP4Nagios.tar.gz
```

With the code downloaded and expanded, move on to compiling and installing.

### Step 3- Compiling and Installing PNP4Nagios

Compile and install the downloaded PNP4Nagios code.

``` BASH
cd PNP4Nagios-0.6.26
./configure --with-httpd-conf=/etc/apache2/sites-enabled
make all
make install
make install-webconf
make install-config
make install-init
```

The process will look something like this:

![PNP4Nagios make install]({{site.url}}/assets/2019-11-30-install-pnp4nagios/makeInstallPnp4nagiosComplete.png)

### Step 4- Configuring NPCD service

The npcd (Nagios Perfdata C daemon) service needs to be configured to run on startup.  

systemctl daemon-reload
systemctl enable npcd.service
systemctl start npcd.service

At the end of those steps, you should end up with a message like such:

![image]({{site.url}}/assets/2019-11-30-install-pnp4nagios/systemctlEnableNpcd.png)

### Step 5- Restarting Apache

After installing PNP4Nagios-webconf you should restart the Apache web server.

``` bash
systemctl restart apache2.service
```

### Step 6- Modifying the `nagios.cfg` file

Next change the `nagios.cfg` file to direct Nagios to process performance data, give it a location to store both `host` and `service` data, define a format for that data and specify the command that will be used to process that data.  We're telling Nagios to process the data in **bulk mode**, it will write the data to temp files then execute a command at an interval to send the data to RRD databases.

```BASH
sudo nano /usr/local/nagios/etc/nagios.cfg
```

Find and change the line:

```BASH
process_performance_data=0
```

to

```BASH
process_performance_data=1

host_perfdata_file=/usr/local/PNP4Nagios
/var/host-perfdata
host_perfdata_file_template=DATATYPE::HOSTPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tHOSTPERFDATA::$HOSTPERFDATA$\tHOSTCHECKCOMMAND::$HOSTCHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$
host_perfdata_file_mode=a
host_perfdata_file_processing_interval=15
host_perfdata_file_processing_command=process-host-perfdata-file-bulk-npcd

service_perfdata_file=/usr/local/PNP4Nagios
/var/service-perfdata
service_perfdata_file_template=DATATYPE::SERVICEPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tSERVICEDESC::$SERVICEDESC$\tSERVICEPERFDATA::$SERVICEPERFDATA$\tSERVICECHECKCOMMAND::$SERVICECHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$\tSERVICESTATE::$SERVICESTATE$\tSERVICESTATETYPE::$SERVICESTATETYPE$
service_perfdata_file_mode=a
service_perfdata_file_processing_interval=15
service_perfdata_file_processing_command=process-service-perfdata-file-bulk-npcd
```

Save and exit the `nagios.cfg` file.  That gives Nagios a place to put the data temporarily and specifies the command to process the data, next configure the command that will do that processing.

### Step 7- Defining Nagios Commands

Nagios will need to commands to execute when `process-service-perfdata-file-bulk-npcd` and `process-host-perfdata-file-bulk-npcd` are called.  Edit the `commands.cfg` file to do that.

``` BASH
sudo nano /usr/local/nagios/etc/objects/commands.cfg
```

Add this text at the bottom of the file

``` BASH
################################################################################
#
# PNP4Nagios
 COMMANDS
#
# Added <date> <initials>
#
#################################################################################

define command {
    command_name    process-service-perfdata-file-bulk-npcd
    command_line    /bin/mv /usr/local/PNP4Nagios

    /var/service-perfdata /usr/local/PNP4Nagios

    /var/spool/service-p$
}

define command {
    command_name    process-host-perfdata-file-bulk-npcd
    command_line    /bin/mv /usr/local/PNP4Nagios

    /var/host-perfdata /usr/local/PNP4Nagios

    /var/spool/host-perfdat$
}
```

Save and exit the `commands.cfg` file.  We'll test the config and restart Nagios so this all starts happening.

### Step 8- Verifying the Nagios Configuration

That's a lot of changes to the Nagios configuration, so test the changes.

``` BASH
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

Which should produce the following:

![image]({{site.url}}/assets/2019-11-30-install-pnp4nagios/checkNagiosConfiguration.png)

If things are OK restart the Nagios service

``` BASH
sudo systemctl restart nagios.service
```

With these changes, there *should* be a bunch of data starting to get generated.  Is that happening?  At this point, we don't know.  Let's check.

### Step 9- Verifying PNP4Nagios is operating

With the changes made, Nagios should be writing temporary files that the `npcd` service is converting into RRD files.  Confirm that.

``` BASH
cd /usr/local/PNP4Nagios
/var/perfdata/localhost/
ls
```

You may see few files if you do this too  quickly.  Which will look something like this:

![image]({{site.url}}/assets/2019-11-30-install-pnp4nagios/checkPNP4NagiosOperation.png)

If you wait(ed) a little longer you'll see more.  Like this:

![image]({{site.url}}/assets/2019-11-30-install-pnp4nagios/checkPNP4NagiosOperationComplete.png)

Bottom line, a nice mix of `.rrd` and `.xml` files should be in this directory.  What's in all of those little fellas'?  Let's find out.

Use a web browser to go to `http://<your nagios host>/PNP4Nagios`.  Use your nagios admin user credentials and you can see this:

![image]({{site.url}}/assets/2019-11-30-install-pnp4nagios/pnp4nagiosWebGUI.png)

You can see at the bottom of the screen that this environment tests page should be removed.  Do just that next.

### Step 10- Removing `install.php` file

Once everything is set with the PNP4Nagios install and config, you'll want to see it display the data and visualizations it is producing.  You can do that by getting rid of the `install.php` file.

``` BASH
$ sudo rm -f /usr/local/PNP4Nagios/share/install.php
```

Return to `http://<your nagios host>/PNP4Nagios` and refresh the browser and you will see something similar to this:

![image]({{site.url}}/assets/2019-11-30-install-pnp4nagios/PNP4NagiosWebGuiHostData.png)

This will default to the first host in your host list, to see another change

`http://<your_nagios_host>/PNP4Nagios/graph?host=<first_host_in_the_list>`

to

`http://<your_nagios_host>/PNP4Nagios/graph?host=<the host you want to see>`

So this is great, data is being saved, processed and passable graphs are being written.  Still, wouldn't it be great if you could get to these pages from within the Nagios GUI?

### Step 11- Modifying the `templates.cfg` file

Each **host** and **service** in Nagios can be easily modified to display their corresponding PNP4Nagios graphs.  To do this, modify the `templates.cfg` file.

``` BASH
sudo nano /usr/local/nagios/etc/objects/templates.cfg
```

At the bottom of the file add:

``` BASH
#######################################################################################
#
# PNP4Nagios
 TEMPLATES
#
# Added <DATE> <MODIFIED BY>
#
#######################################################################################

define host {
   name       host-pnp
   action_url /PNP4Nagios/index.php/graph?host=$HOSTNAME$&srv=_HOST_
   register   0
}

define service {
   name       service-pnp
   action_url /PNP4Nagios/index.php/graph?host=$HOSTNAME$&srv=$SERVICEDESC$
   register   0
}
```

Don't close the file just yet, this is only the first part of integrating the PNP4Nagios visualizations.

### Step 12- Modifying the Generic Host and Service definitions

Now that you have the templates created, direct the **hosts** and **services** to use them.

``` BASH
sudo nano /usr/local/nagios/etc/objects/templates.cfg
```

Find the `Generic host definition template` and add the line `use    host-pnp    ;Added <date><modifier>` so that it looks something like this:

![image]({{site.url}}/assets/2019-11-30-install-pnp4nagios/genericHostModification.png)

Do the same with the  `Generic service definition template` and add the line `use    host-pnp    ;Added <date><modifier>` so that it looks something like this:

![image]({{site.url}}/assets/2019-11-30-install-pnp4nagios/genericServiceModification.png)

Great!  Let's test and reload Nagios.

### Step 13- Testing the new Nagios configuration and restarting

Not that one would make a mistake in that many text file changes, but let's be on the safe side.

``` BASH
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

Cross fingers, and hope to see something like this.

![image]({{site.url}}/assets/2019-11-30-install-pnp4nagios/checkNagiosConfiguration.png)

If things are OK restart the Nagios service

``` BASH
sudo systemctl restart nagios.service
```

Let's look at the results of all of this work on the Nagios GUI.

### Step 14- Viewing the performance graphs integrated into the Nagios GUI

All of the previous changes should make the PNP4Nagios visualizations easily accessible from the Nagios GUI>

Go to the Nagios Web interface `http://<your_nagios_host>/nagios` and use your credentials to login.  Click `Curent Status | Hosts`.

You'll notice a new icon by each host:

![image]({{site.url}}/assets/2019-11-30-install-pnp4nagios/PNP4NagiosIntegratedNagiosGUI.png)

Click that and you will see graphs for each service recorded for the host.

![image]({{site.url}}/assets/2019-11-30-install-pnp4nagios/PNP4NagiosIntegratedNagiosGUIDetail.png)

Dig yourself!  You've made Nagios much more useful.

## Conclusion

By installing and configuring PNP4Nagios you've enabled saving historical performance monitor data in Nagios.  You've also integrated some serviceable though not particularly attractive visualizations.

Creating this database linkage allows you to up the visualization game if you want.  Specifically, it's the gateway to using `Grafana` to build dashboards from the Nagios data.

### Resources

* <https://support.nagios.com/kb/article/nagios-core-performance-graphs-using-pnp4nagios-801.html#Debian>
* <https://docs.pnp4nagios.org/>

### Revision History

* Writing begins- 11/30/2019
* Content review- 6/16/2020
* Style Guide review- 6/16/2020
* Published- 6/16/2020
