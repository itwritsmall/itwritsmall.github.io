---
title: Title
date: 2021-09-11 00:00:00 Z
categories:
- blog
tags:
- tag
layout: post
summary: How to install OpenSSH Server on Hyper-V Server 2019.
---

How to install OpenSSH Server on Hyper-V Server 2019.

### Introduction
Hyper-V Server 2019 installs without a Desktop interface.  There is a rudimentary configuration program called ??? which makes me nostalgic for the Netware console.  A more sophisticated web interface, Windows Admin Center, also exists and it will get you through most administrative tasks.  

Within this remote lab we're going to be editing a lot of text files and running a lot of PowerShell scripts (Automated Lab specifically) so it makes sense to me to just connect Visual Studio Code to the Hyper-V Server.  This can be done, but only if the Hyper-V server is running the OpenSSH server.  THis post will walk you through installing (and testing) OpenSSH.

## Goals
* Install OpenSSH Feature
* Set the service to start automatically
* Ensure there is a firewall rule to allow traffic

### Prerequisites
* Hyper-V Server 2019
* PowerShell prompt, admin level (you can do this locally or remotely using Remote Desktop)

## Install OpenSSH Server
Hyper-V 2019 Server has the OpenSSH Client installed, but not the server.  We can use Powershell to do this.  

To find the package and see if it is installed type:

``` powershell
Get-WindowsCapability -Online -Name Open*
```

You will see something similar to this:

![image]({{site.url}}assets/2021-09-11-hypervServer2019-install-openSSH/get-windowscapability.png)

The client is installed, but not the server.  Let's change that.

``` powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

While you're cursing whoever put four tildes into the file name, you'll see something like this:

![image]({{site.url}}assets/2021-09-11-hypervServer2019-install-openSSH/add-windowscapability-progress.png)

And then...

![image]({{site.url}}assets/2021-09-11-hypervServer2019-install-openSSH/add-windows-capability-error.png)

Which gives us something else to curse.  Let's open a Powershell console that has the correct rights to do the work we need done.  The only way I could figure to do this was to remote desktop (mstsc.exe) into the server.

From the command prompt (```cmd.exe```), launch PowerShell

``` console
powershell -Command "Start-Process PowerShell -Verb RunAs"
```

![image]({{site.url}}assets/2021-09-11-hypervServer2019-install-openSSH/launch-elevated-powershell.png)

And this launched a PowerShell prompt with administrator rights.  It occured to me that doing this from Windows Admin center might do the same thing, there was just no way that I knew of to switch into that new console.

At this point run

``` powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

And you'll have more success.

![image]({{site.url}}assets/2021-09-11-hypervServer2019-install-openSSH/add-windows-capability-progress2.png)

and eventually...

![image]({{site.url}}assets/2021-09-11-hypervServer2019-install-openSSH/add-windowscapability-success.png)

## Start Service and Set Startup

This installs the OpenSSH server as a service, but it doesn't start it or set it to start automatically.  Let's stay in this elevated Powershell console.

If you'd like to prove to yourself that the service isn't running

``` powershell
Get-Service -Name sshd
```
And you shall see:

![image]({{site.url}}assets/2021-09-11-hypervServer2019-install-openSSH/get-service-sshd-stopped.png)

You really need to be more trusting.  Let's start this fella' up.

``` powershell
Start-Service sshd
```

And then set the service to start auto-magically.

```powershell
Set-Service -Name sshd -StartupType 'Automatic'
```

There's zero feedback from either of these commands, so you can confirm with an enhanced:

``` powershell
get-service -name sshd | select-object -property *
```

which nets the more informative:

![image]({{site.url}}assets/2021-09-11-hypervServer2019-install-openSSH/enhanced-get-service-sshd.png)

You may also want to check that a corresponding firewall rule has ben created with the service.  To do this:

``` powershell
Get-NetFirewallRule -Name *ssh*
```

Will get you:

![image]({{site.url}}assets/2021-09-11-hypervServer2019-install-openSSH/get-netfirewallrule.png)


## Conclusion
This gets you a running OpenSSH server on Hyper-V Server 2019.  The purpose of this exercise is to then use this service to allow Visual Studio Code to connect to the server to manage and run Automated Lab.  The instructions for that next task are here:

### Resources
* reference- url

### Revision History
* 9/11/2021- Writing begins
* Content review-
* Style Guide review-
* Published-