---
layout: blog
title: "How To Set Up PowerCLI, PowerShell and Visual Studio Code on Ubuntu Desktop 18.04"
categories: [blog]
tags: [PowerCLI, vSphere, Ubuntu, PowerShell, Visual Studio Code, Git]
summary: How to confiugre a vSphere PowerCLI development environment on Ubuntu Linux Desktop using Microsoft PowerShell and Visual Studio along with GitHub.
---

## Introduction

VSphere PowerCLI can be used to automate the management of nearly all elements of a VMWare infrastructure.  PowerCLI is a PowerShell add-on and PowerShell can be installed on Windows or Linux.  This post will go the Linux route and install PowerShell 7 and PowerCLI 12 onto Ubuntu Desktop 18.04.  Also available for Windows and Linux is Visual Studio Code, which will serve as an IDE for the combo.

> **Note:**  There isn't official support for PowerShell on Ubuntu 20.04 Desktop yet, though it is available as a Snap package.

## Goals

1. Install Visual Studio Code onto Ubuntu Desktop 18.04.
1. Install PowerShell 7 onto Ubuntu Desktop 18.04
1. Install the PowerCLI 12 module in PowerShell
1. Install extensions for PowerShell in Visual Studio Code
1. Access the VMWare environment

## Prerequisites

- A virtual or physical machine with Ubuntu 18.04 Desktop installed
- A VMWare environment.

### Ubuntu 18.04 Desktop

I'm using the 64-bit edition, not sure if it matters but I'm certain I'm not going to spend time finding out.  Be good to yourself and get the extra 32 bits.  It's version 18.04.4 LTS.

### VMWare

You'll need a VMWare environment or what is the point of all of this?  If you are using the Free edition of ESXi, PowerCLI will be limited to read only capabilities.  A licensed vCenter installation gets you the ability to also write.

## Tutorial

- Step 1- Installing Visual Studio Code
- Step 2- Installing and Testing PowerShell 7
- Step 3- Installing and Testing PowerCLI Module(s)
- Step 4- Installing the Visual Studio Code PowerShell Extension
- Step 5- Installing Git (optional)

### Step 1- Installing Visual Studio Code

Start with installing Visual Studio Code.  You can install from the command line using `apt` or `snap` but for the purposes of this tutorial, we'll go the GUI route and use the Ubuntu Software Manager.

Click the Show Applications icon.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/showApplications.png)

Double click the Ubuntu Software icon.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/ubuntuSoftware.png)

When that has launched, click the Search icon.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/ubuntuSoftwareSearch.png)

Type in `visual studio code` and hit enter.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/ubuntuSoftwareSearchVSCode.png)

The results should include the Visual Studio Code application.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/ubuntuSoftwareSearchReult.png)

Double click that to get details about the program and click the **Install** button.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/ubuntuSoftwareVSCodeInstall.png)

You will get a prompt to provide elevated credentials.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/ubuntuAuthenticationRequired.png)

Once that's provided the program will install.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/VSCodeInstallProgress.png)

When the install is complete you can close the Ubuntu Software application.  We'll launch VS Code next and do the rest of the installs from there.

### Step 2- Installing and Testing PowerShell 7

We'll do the rest of the work from Visual Studio Code, so launch that program.  Once you're in, select **New Terminal** from the **Terminal** menu.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/VSCodeNewTerminalCommand.png)

This will open a new BASH terminal at the bottom of the Visual Studio Code window.  
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/VSCodeBlankTerminal.png)

Install Microsoft's GPG keys.

``` BASH
wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb
```

This should finish without feeback.

Register the keys.

``` BASH
sudo dpkg -i packages-microsoft-prod.deb
```

This will return something similar to this.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/PowerShellRegisterMicrosoftGPGKeys.png)

Update the package cache.

``` BASH
sudo apt-get update
```

This will produce something like this, you want to see that `packages.microsoft.com` is included.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/PowerShellUpdatePackageCache.png)

Install the PowerShell 7 Package.

``` BASH
sudo apt-get install powershell
```

This will result in something like this.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/PowerShellInstall.png)
Type `y` to proceed.

You will see some progress.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/PowerShellInstallProgress.png)
Once the install is done and you are back at the command prompt test hte install.  Type `pwsh`.  
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/PowerShellPrompt.png)
You'll see the version of PowerShell that's running and that the prompt has changed to `PS <current directory>`.  You can also see the that terminal type has changed within Visual Studio Code like such:
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/VSCodePowerShellTerminal.png)
Stay in the PowerShell environment and let's move on to installing PowerCLI.

### Step 3- Installing and Testing PowerCLI Module(s)

We've got our IDE up and running, we've got PowerShell installed so it's time to get PowerCLI into the mix.  At the PowerShell prompt (or type `pwsh` to get back in).  Grab the PowerCLI module from the PSGallery by typing

``` powershell
Install-Module VMWare.PowerCLI
```

Which should produce the following warning:
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/PowerShellPSGalleryWarning.png)

Type `A` to move past this and install the module.  You'll see some progress communications.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/PowerCLIInstallProgress.png)

When the install is finished, type

``` powershell
Get-Module ListAVailable VMware*
```

You'll see that it's more accurate to refer to PowerCLI as modules instead of a single module as **a lot** got installed.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/PowerCLIListModules.png)

Seems appropriate to finish testing the install by connecting to your VMWare environment.  Assuming your environment uses a self-signed certificate, you'll need to set PowerCLI to ignore certificate warnings.

``` powershell
Set-PowerCLIConfiguration -InvalidCertificateAction:ignore
```

Which will result in a warning like such:
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/PowerCLIInvalidCertificateActionWarning.png)

Type `A` to accept, the results will look similar to:
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/PowerCLIInvalidCertificateActionResults.png)

Now that this business is complete, let's connect to VMWare.

``` powershell
Connect-VIServer <FQDN of VCenter Host or ESXI host>
```

Connecting to an ESXi host gets you something like this.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/PowerCLIConnectToESXi.png)
Connecting to a VCenter host gets you something like this.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/PowerCLIConnectToVCenter.png)

To test a little further, type `get-vm` and you'll see a list of the VMs on the host.  This works if you are connected to the ESXi host **or** a VCenter host.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/PowerCLIGet-VMResults.png)
Next, add some extensions to VSCode to make it easier to work with PowerShell and PowerCLI.  I'll be along after I figure out why I assigned some VMs 2 CPUs and others just 1.

### Step 4- Installing the Visual Studio Code PowerShell Extension

First, in Visual Stduio Code, install an extension to make working with PowerShell code easier.  To do this, click on the extenstions marketplace icon.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/VSCodeExtensionsMarketplaceIcon.png)

In the Search box, type `ms-vscode.powershell`, which should find Microsoft's PowerShell extension.
![image]({{site.url}}/assets/2020-05-25-setup-powercli-ubuntu/VSCodeSearchPowerShellExtension.png)
Click Install.

### Step 5- Installing Git (Optional)


## Conclusion
		○ Summary
		○ What to do next
### Resources
* reference- url

### Revision History
* 05/25/2020- The writing begins
* Content review-
* Style Guide review-
* Published-