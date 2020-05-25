---

layout: blog
title: "Set Up PowerCLI on Ubuntu Desktop"
categories: [blog]
tags: [PowerCLI, vSphere, Ubuntu, Powershell, Visual Studio Code, Git]
summary: How to create a vSphere PowerCLI development environment on Ubuntu Linux Desktop.
---
*How to confiugre a vSphere PowerCLI development environment on Ubuntu Linux Desktop.*

### Introduction

VSphere PowerCLI can be used to automate nearly all elements on VMWare infrastructure.  PowerCLI is a Powershell add-on and Powershell can now be installed on Windows or Linux.  We'll go the Linux route and install onto Ubuntu Desktop.  Other elements that will make up the development environment will include Microsoft Visual Studio Code and Git.

## Goals

1. Install test Powershell
1. Install and test PowerCLI
1. Install Visual Studio Code and plug-ins
1. Install Git
1. Create a GitHub repository
1. Synchronize the GitHUb rpository between the development machine and the cloud

## Prerequisites

1. A virtual or physical machine with Ubuntu 18.04 Desktop installed
1. A VMWare environment running VCenter
1. A GitHub account

### Ubuntu 18.04 Desktop

I'm using the 64-bit edition, not sure if it matters but I'm certain I'm not going to spend time finding out.  Be good to yourself and get the extra 32 bits.  It's version 18.04.4 LTS.

### VMWare

You'll need a VMWare environment or what is the point of all of this?  If you are using the Free edition of ESXi, PowerCLI will be limited to read only capabilities.  A licensed vCenter installation gets you the ability to write.

### Github

So technically, this isn't a requirement but you're installing PowerCLI to write code and if you're writing code you should be managing versioning.  Github will get you there, a free account will work for your purposes.

## Tutorial

- Step 1- Installing Powershell

### Step 1- Installing Powershell



## Step 1,2….N
		○ Step # - <-ing word> 
		○ Introductory sentence
			§ Description of step/code
			§ Code block
			§ Resulting image (opt)
		○ Transition sentence
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