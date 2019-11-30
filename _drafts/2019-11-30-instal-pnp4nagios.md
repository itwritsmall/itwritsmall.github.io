---

layout: post
title: "Install PNP4Nagios on Debian and Nagios Core"
categories: [Tutorial]
tags: [Nagios, RRDTool, PNP4Nagios, Debian]
excerpt: How to install PNP4Nagios and RRDTool for Nagios on Debian Linux.
---

How to install PNP4Nagios and RRDTool for Nagios on Debian Linux.

### Introduction
What is the goal, what software is involved, benefits/reasons to follow the tutorial

## Goals

## Prerequisites

### Prerequisite 1...N
Hardware required, other software, required DNS or SSL, additional user accounts

## Step 1,2….N
		○ Step # - <-ing word> 
		○ Introductory sentence
			§ Description of step/code
			§ Code block
			§ Resulting image (opt)
		○ Transition sentence
## Step 1- Install RRDTool
``` BASH
$ sudo apt-get install -y rrdtool librrds-perl php-gd php-xml
```
## Step 2- Download pnp4Nagios Source
``` BASH
$ cd ~
$ sudo wget -O pnp4nagios.tar.gz https://github.com/lingej/pnp4nagios/archive/0.6.26.tar.gz
$ sudo tar xzf pnp4nagios.tar.gz
```
## Step 3- Compile and Install pnp4Nagios

``` BASH
cd pnp4nagios-0.6.26
./configure --with-httpd-conf=/etc/apache2/sites-enabled
make all
make install
make install-webconf
make install-config
make install-init
```

http://<your nagios host>/pnp4nagios
/etc/apache2/sites-enabled/pnp4nagios.conf
nagios user/group nagios nagios
/usr/local/pnp4nagios

### Step 4- Configure NPCD service
systemctl daemon-reload
systemctl enable npcd.service
systemctl start npcd.service

### Step 5- Restart Apache
After installing pnp4nagios-webconf you should restart the Apache web server.
systemctl restart apache2.service

### Step 6- Modify the `nagios.cfg` file
```BASH
$ sudo nano /usr/local/nagios/etc/nagios.cfg
```
Find and change the line:
```BASH
process_performance_data=0
```
to
```BASH
process_performance_data=1

host_perfdata_file=/usr/local/pnp4nagios/var/host-perfdata
host_perfdata_file_template=DATATYPE::HOSTPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tHOSTPERFDATA::$HOSTPERFDATA$\tHOSTCHECKCOMMAND::$HOSTCHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$
host_perfdata_file_mode=a
host_perfdata_file_processing_interval=15
host_perfdata_file_processing_command=process-host-perfdata-file-bulk-npcd

service_perfdata_file=/usr/local/pnp4nagios/var/service-perfdata
service_perfdata_file_template=DATATYPE::SERVICEPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tSERVICEDESC::$SERVICEDESC$\tSERVICEPERFDATA::$SERVICEPERFDATA$\tSERVICECHECKCOMMAND::$SERVICECHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$\tSERVICESTATE::$SERVICESTATE$\tSERVICESTATETYPE::$SERVICESTATETYPE$
service_perfdata_file_mode=a
service_perfdata_file_processing_interval=15
service_perfdata_file_processing_command=process-service-perfdata-file-bulk-npcd
```

Save and exit the `nagios.cfg` file.

### Step 7- Define Nagios Commands

``` BASH
$ sudo nano /usr/local/nagios/etc/objects/commands.cfg
```
Add this text at the bottom of the file

``` BASH
################################################################################
#
# PNP4NAGIOS COMMANDS
#
# Added <date> <initials>
#
#################################################################################

define command {
    command_name    process-service-perfdata-file-bulk-npcd
    command_line    /bin/mv /usr/local/pnp4nagios/var/service-perfdata /usr/local/pnp4nagios/var/spool/service-p$
}

define command {
    command_name    process-host-perfdata-file-bulk-npcd
    command_line    /bin/mv /usr/local/pnp4nagios/var/host-perfdata /usr/local/pnp4nagios/var/spool/host-perfdat$
}
```
Save and exit the `commands.cfg` file

### Step 8- Verify the Nagios Configuration
``` BASH
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```
If things are OK restart the Nagios service

``` BASH
$ sudo systemctl restart nagios.service
```
### Step 9- Verify PNP4Nagios is operating
``` BASH
cd /usr/local/pnp4nagios/var/perfdata/localhost/
```
``` BASH
ls
```
You may see a few files if you do this quickly.

If you wait(ed) a little longer you'll see more.

You can also use a web browser to go to `http://<your nagios host>/pnp4nagios`.  Use your nagios admin user credenitals and you can see this:

### Step 10- Remove `install.php` file
``` BASH
$ sudo rm -f /usr/lcoal/pnp4nagios/share/install.php
```
Return to `http://<your nagios host>/pnp4nagios` and refresh the browser and you will see something similar to this:

This will default to the first host in your host list, to see another change
`http://http://<your_nagios_host>/pnp4nagios/graph?host=<first_host_in_the_list>` to `http://http://<your_nagios_host>/pnp4nagios/graph?host=<the host you want to see>`

### Step 11- Modify the `templates.cfg` file
``` BASH
$ sudo nano /usr/local/nagios/etc/objects/templates.cfg
```
At the bottom of the file add:
``` BASH
#######################################################################################
#
# PNP4NAGIOS TEMPLATES
#
# Added <DATE> <MODIFIED BY>
#
#######################################################################################

define host {
   name       host-pnp
   action_url /pnp4nagios/index.php/graph?host=$HOSTNAME$&srv=_HOST_
   register   0
}

define service {
   name       service-pnp
   action_url /pnp4nagios/index.php/graph?host=$HOSTNAME$&srv=$SERVICEDESC$
   register   0
}
```
### Step 12- Modify the Generic Host and Service definitions
``` BASH
$ sudo nano /usr/local/nagios/etc/objects/templates.cfg
```

Find the `Generic host definition template` and add the line `use    host-pnp    ;Added <date><modifier>` so that it looks something like this:


Do the same with the  `Generic service definition template` and add the line `use    host-pnp    ;Added <date><modifier>` so that it looks something like this:

### Step 13- Test the new Nagios configuration and Restart

``` BASH
$ sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```
If things are OK restart the Nagios service

``` BASH
$ sudo systemctl restart nagios.service
```

### Step 14- See the performance graphs integrated into the Nagios GUI

Go to the Nagios Web interface `http:<your_nagios_host>\nagios"` and use your credentials to login.  Click `Curent Status` | `Hosts`.

You'll notice a new icon by each host:

Click that and you will see graphs for each service recorded for the host.





## Conclusion
		○ Summary
		○ What to do next
### Resources
* reference- url
* https://support.nagios.com/kb/article/nagios-core-performance-graphs-using-pnp4nagios-801.html#Raspbian


### Revision History
* 11/30/2019- Writing begins
* Content review-
* Style Guide review-
* Published-