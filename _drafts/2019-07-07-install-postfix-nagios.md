---

layout: blog
title: "Installing Postfix for Nagios using Office365"
categories: [tutorial]
tags: [nagios, postfix, office365, mailx, raspbian]
excerpt: How to install Postfix with Nagios and Office365 on Raspbian.
---


## Introduction
Once you have installed Nagios and have it configured for simple monitoring a logical next step is to configure e-mail alerting.   

## Goals
In this tutorial, you'll install the Postfix e-mail server and configure it to use Office365 to send alet messages within your organization.

## Prerequisites

### Office365
In this tutorial, I'll use Office365 as the e-mail server.  I'm guessing that using another service (GMail, etc.) is trivial but the configuration details will be different.

### Nagios
You'll need a working installation of Nagios running.  Here's my tutorial to accomplish that:  http://www.itwritsmall.com/2019-03-09-install-nagios-raspbian.html

## Tutorial

### Step 1- Install Postfix
You'll install `postfix` to serve as a mail server and `mailtutils` to serve as a mail client.  Nagios will use the `mailutils` client to create the e-mail message, the mail client will use the `postfix` server to send the message to Office365.  To install the programs, open a terminal session on you Nagios server and type:

``` bash
$ sudo apt-get install postfix
$ sudo apt-get install mailutils
```

![apt-get install postfix]({{site.url}}/assets/2019-03-12-install-postfix-nagios/aptGetInstallPostfix.png)

Choose Internet Site, that's the most appropriate configuration for outbound mail only.

`sudo nano /etc/postfix/main.cf`

` relayhost = [smtp.office365.com]:587

confirm `mynetworks = 127.0.0.0/8 

change `inet_interfaces = loopback-only

``` bash
smtp_use_tls = yes
smtp_always_send_ehlo = yes
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_sasl_tls_security_options = noanonymous
smtp_tls_security_level = encrypt
smtp_generic_maps = hash:/etc/postfix/generic
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
```

`sudo chown root:root /etc/postfix/sasl_passwd
`sudo chmod 0600 /etc/postfix/sasl_passwd
`sudo postmap /etc/postfix/sasl_passwd

` sudo nano /etc/postfix/generic

` root@localdomain <user@domain.com>
` @localdomain <user@domain.com>

``` bash
sudo chown root:root /etc/postfix/generic
sudo chmod 0600 /etc/postfix/generic
sudo postmap /etc/postfix/generic
```

`sudo service postfix restart

Test 

` echo "Testing, testing, testing" | mail -s "Testing Postfix" <recipient address> -a "FROM:<sender address>"


In the file `/usr/local/nagios/etc/objects/command.cfg` append `notify-host-by-email` and `notify-service=by-email` with

``` bash 
 /usr/bin/mail -s "** $NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$ **" $CONTACTEMAIL$ -r sender@example.com
```

Give Nagios someone to send to

``` bash
 sudo nano /usr/local/nagios/etc/objects/contacts.cfg
```

``` bash
define contact {
        contact_name                            Contact1
        alias                                   ContactNameAlias
        email                                   email-address
        service_notification_period             24x7
        service_notification_options            w,u,c,r,f,s
        service_notification_commands           notify-service-by-email
        host_notification_period                24x7
        host_notification_options               d,u,r,f,s
        host_notification_commands              notify-host-by-email
}

```

Add the user to the `nagiosadmins` group defined below it

We will use Office365 as the relay to send messages to mailboxes in your Office365 organization.  This is the easiest way to set up e-mail alerts but it prevents you from sending to users outside the organization.

`smtp.office365.com

troubleshooting 
` cat /var/log/mail.log

Use web gui in Nagios Reports | Event Log

## Conclusion
		○ Summary
		○ What to do next
### Resources
* https://gordan.jandreoski.me/how-to-configure-postfix-relay-to-office365-on-ubuntu-14-04/


### Revision History
* 02/18/2019- Writing begins
* Content review-
* Style Guide review-
* Published-