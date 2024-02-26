---
title: Building an Automated, Virtual Lab
date: 2021-09-11 00:00:00 Z
categories:
- blog
tags:
- Automated Lab
- Windows Server
layout: post
summary: Building an evaluation environment using (mostly) free software on (mostly)
  consumer grade hardware.
---

Building an evaluation environment using (mostly) free software on (mostly) consumer grade hardware.

### Building an Automated, Virtual Lab

I've started doing more product evaluation as part of my job.  At first I would install the software on the same VM that I was using to do my regular work for the company.  Never a good idea.

Occasionally I would spin up an additional VM on my laptop and while this got me an isolated environment it took a lot of time to set up.  It also took (limited) resources that I was trying to use to get work done.

## Dedicated Hardware

Fortunately, I have a desktop that I inherited when my last job ended.  Also passed on to me were some additional storage and memory really making it a capable machine.

I had loaded VMWare onto the machine and was hosting a couple of VMs there.  I have some VMWare experience (and have a VMUG subscription) so my thought was I would write some PowerCLI code to build and tear down isolated lab environments.

## Automated Lab and Microsoft Server

The more I thought about that, the more I realized that would be a big coding job.  Searching around for an existing project, I came across the Automated Lab scripts.

This moved me off of the discounted VMWare platform but Microsoft offers it's Windows Server as a free Hypervisor version.  That leaves licensing the VMs, but the eval versions of Microsoft products should suffice for my purposes.  None of these environments really need to be running more than a month or two.

## Conclusion

If this were a bigger part of my job or demanded more complex configurations, I'd probably use more dev-ops oriented tools to automate this but I believe for my current purposes this is going to work.  I'm going to begin building and using this environment and I'll write about it here.  Stay tuned:

### Resources
* reference- url

### Revision History
* 09/11/2021- Writing begins
* Content review-
* Style Guide review-
* Published-