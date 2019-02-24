---

layout: post
title: "How to set up a web site using Jekyll on Windows Subsystem for Linux (WSL)"
categories: [tutorial]
tags: [windows10, jekyll, git, github]

---

# How to set up a web site using Jekyll on Windows Subsystem for Linux (WSL)

### Introduction

I started prototyping this site using WordPress but it quickly became apparent to me that it was going to be tough to achieve what I was hoping to do on that platform.  My requirements are:
* Ability to use whatever editor I prefer at the moment.
* Support for multiple operating systems.
* Hosting flexibility with some free options.  
* A platform that is light weight for what I intend to be a "low-design" site.
* The site should be portable and easy to back up.

To do this, I decided to use Jekyll to manage the web site.  Jekyll is a Ruby program that does the work of applying (Liquid) template content and settings to markup documents and turning them into static html files that can be served up.

This tutorial covers the following software:
* **Windows Subsystem for Linux (WSL) on Windows 10**- While Jekyll makes it possible to use nearly any operating system, Windows is my OS of choice.  Still, I wanted to use the supported Linux based Jekyll tools.
* **Jekyll**- For converting markup documents to a static HTML site.  Related to Jekyll: **Ruby**, **Blender**, and **Rouge**.
* **Git and GitHub**- For source control and as a free code repository and hosting option.
* **Poole**- As a simple Jekyll theme to start with.
* **Visual Studio Code**- Your choice of text editor will work, my choice is Visual Studio Code.

This tutorial will explain how to get the software installed on two computers and get an initial web site set-up and serving your web pages.  There are a number of optional steps that you can easily skip if you're planning on a different work flow.

## Prerequisites
I'll assume that you have a few things set up already.  Specifically:
* **Windows 10**- Any version (Home, Professional, Enterprise) will work.
* **Git Hub Account**- "Individual- Free" will do you, but it should work on any pricing plan.
* **Visual Studio Code**- Or your choice of text editor.
* **Knowledge of Markup or HTML**- you will need to know your way around a markup language to create the files in.  I'll use markup here.
* **An Idea**- Optional, but why set up a web site if you don't have something to share?

## Step 1: Install Windows Subsystem for Linux
The Windows Subsystem for Linux feature is not installed by default and needs to be enabled.  To do this, open the "Turn Windows features on or off".  Press the Start key and type `add feature` in the search box.
![alt text]()

Choose **Turn Windows features on or off** from the search results to launch the Control Panel.
![alt text]()

Scroll down in the control panel and check on the **Windows Subsystem for Linux** feature.  
![alt text]()

Click **OK** and the feature will install.  Click **Restart Now** when the installation is complete.
![alt text]()

You will need to choose a Linux Distribution next.  For this tutorial, I chose Ubuntu 18.04LTS.  You can use this URL https://www.microsoft.com/en-us/p/ubuntu/9nblggh4msv6?rtc=1&activetab=pivot:overviewtab.  Click Get to launch the Microsoft Store and click Get again to start the download and the installation.

After the installation completes, click the **Launch** button.  This will start the installation of the Ubuntu Distribution.  As the installation promises, you've got a couple of minutes to kill.

The last step of the installation is setting a user name and password.  Wish this was tied into the machine/domain login...but it's separate so make sure to set a password that you will remember.

Once the install process is complete, hit the start button and type `cmd` in the search field.  When the command prompt opens type `bash` and you will be placed in the BASH shell for Ubuntu.

First thing, update the repositories by typing `sudo apt-get update`.  The apt-get program will work through the repository list and get the most current programs.

> I purposely don't add the `-y` switch to commands ibn the tutorial so that you can see what will happen before approving it.

Next, perform an upgrade to bring the Ubuntu software up to current.  Initiate this by typing `sudo apt-get upgrade`.

## Step 2: Install Ruby

We've got a upgraded Ubuntu installation now, so let's get to doing the work of installing Jekyll.  Install Ruby and build-essential by typing `sudo apt-get install build-essential ruby ruby-dev`.  

## Step 3: Install Jekyll
With Ruby installed, you can install Jekyll which is a Ruby "Gem".  Still in the BASH shell, type `sudo gem update` and then `sudo gem install jekyll`

You may get a Windows Defender Firewall dialog asking you to allow Ruby access to the network.

You can check that everything worked by typing `jekyll -v`.

Once the installation is complete, it's time to set up a repository on GitHub to manage the source code for the site.

## Step 4: Set up a Repository
You *could* skip using source control entirely and just use something like OneDrive to replicate the site files between two (or more) machines.  Git and GitHub are useful tools for managing source code for more than just web sites though, so I'll go through the process here.

Open a web browser and go to GitHub (https://github.com/).  Log in using the account appropriate for the website you want to create.

Create a new repository.  If you want to serve the site from GitHub then set the repository as Public, choose private if you're just going to be using GitHub to provide source control.  Do NOT choose  `Initialize this repository with a README`.  We'll use Jekyll to add the initial content.

> If you're going to be using GitHub to host your pages then the repository name needs to be the same as your GitHub user name.  TEST TO SEE IF THIS IS THE CASE!!

Click the `Create repository` button.

The next screen will a display a URL for the repository.  Leave the window open, you'll need the URL in the next step where we set up the repository on your Windows machine.

## Step 5: Clone Site Repository
In this step, we'll clone the site repository into a folder on your local machine.  For this tutorial, I will use C:\Users\<user name>\Documents.

At the BASH prompt navigate to the C:\Users\<user name>\Documents folder.  
Now, use the git command to clone the repository to your local machine.  Type `sudo git clone <that URL we talked about earlier>`

The command should complete with the warning that `You appear to have cloned an empty repository` which we already established.  The command will create a folder in your Documents folder with the same name as the repository on GitHub.

> If you navigate to the folder on your Windows machine, you'll find that while the repository is empty, the folder is not.  There's a .git folder in there now.  Make sure that you keep that .git folder intact.

Let's move some stuff to the folder.

## Step 6: Install and Modify Poole Theme
I am no graphic designer, and this site does a good job of proving that.  Instead, I'm going to use the work of the developer of the Poole Theme to build the site.  In a web browser navigate to the Poole project (https://github.com/poole/poole) and download the .zip file. Extract the files from the download and copy them to the folder that was created in step 5.   

This should give you a folder that looks something like this:

What you have is the shell of your new site.  It's got all of the directories and template files to get you going.  Let's make some changes to some files to customize it for your needs.

First up, open the _config.yml in the text editor of your choice.  Change the following to suit your needs:

* Setup | title: <- _set this to the name of your site_
* Setup | tagline: <- _a brief description of your site_
* Setup | url: <- _the url for your web site_
* Setup | baseurl <- _the base url for the pages in your web site.  If you use GitHub pages, this will be "/<your GitHub user name>/"_
* About/contact | name: <- _your name here_
* About/Contact | url: <- _if you've got a different personal web site or a LinkedIN profile, throw that here_
* About/Contact | email: <- _an email alias_

When you've finished your changes, save the file up.

## Step 7: Send Changes to Repository
Let's get the skeleton of your site up to your repository.

Back at the BASH shell, configure a couple of git commands to configure the local repository

First, set the e-mail:

`git config --global user.email "<your e-mail>"`

Next, add the GitHub user account name to the repository:

`git config --global user.name "<your git hub account name>"`

That stuff only needs to be done once for the repository.  Next let's perform the sequence you will use each time you make changes locally and want to push them up to the repository.  The git sequence we will use is add -> commit -> push

First, an optional step.  You can list all of the new and changed files that will be committed by typing:

`git status`

Next, snapshot all of the files to prep them for versioning by typing:

`git add --all`

Now, record the file snapshots into the version history by typing:

`git commit -m "<some info about this first commit>"`

Finally, send the commits from the local repository (origin) to the (master) branch in your GitHub repository.

`git push origin master`

So now, if you look at your GitHub repository you should see the same files as are on your computer.

If you want to host your files on GitHub (or you just want to preview Poole), go into the settings for your repository and scroll down to the GitHub Pages section.

In the Source pulldown, change the setting from None to Master Branch and click Save.  The interface will display a URL where your site is published.

>If the Poole site doesn't display cleanly or links don't work go back and check your baserurl setting from Step 6.

## Step 8: Create a Test Post
If you poked around on the GitHub hosted pages you've seen the sample material included in the Poole theme.  That's cool and everything, but you're probably doing this to get your own message out.  Let's make a post you can call your own.

On your Windows machine, open a text editor of your choosing.  Jekyll can interpret Markdown and HTML documents.  We'll make a quick markup document.

` # Grievance 1`

`People who stop right after exiting a revolving door really chap my ass`

Then save the file into the `<site name>\_posts` directory.  Give it a file name using the following format:
`<YYYY>-<MM>-<DD>-<descriptive-name>.md` as an example `2019-02-21-my-manifesto.md`

> I am finding that for my needs Markup is substituting really well for HTML.  Check the Markdown Cheatsheet in the references section and try it for yourself.

All of this work and we haven't even used Jekyll on the workstation yet.  Let's change that.

## Step 9: Preview the Site
You could, if you wanted to build a site on GitHub never install Git or Jekyll or a single file on your computer.  Installing Jekyll provides a couple of advantages though.  First, you can work on a local version of the site before sending the changes to your production site at git hub.  To do this

`jekyll serve`

> Ruby Gem dependencies error.  

The other advantage of using Jekyll locally is you can build a site of static HTML pages that you can upload to any Web Server.  `jekyll serve` did just that for you in addition to serving up a preview (how nice!).

You can find the files in the `_site` directory.  If you want to create or update those files you can also issue the following command:

`jekyll build`

## Step 10: Send Changes to Repository
Bounce back to Step 7 and send the changes you made up to your repository.
## Step 11: Set Up an Additional Device [Optional]

If you want to set up a second Windows device to edit your site from just perform steps 1,2,3 and 5 on that machine.  From then on you can edit files on that machine and preview or build the site (Step 9) and synchronize your changes with your GitHub repository (Step 7)

When you are using more than one machine (though it's probably good practice with just one machine too) you'll want to make sure you have the latest source code in your local working directory.  From a BASH console in your working directory type:

` git pull `

## Step 12: Publish Site using FTP [Optional]

## Conclusion

### Do This Next
* Use the Rouge code formatter
* Add Google Analytics to your Jekyll Site
### Resources
The following resources were useful in creating this tutorial:
* Blogging Like a Hacker- http://tom.preston-werner.com/2008/11/17/blogging-like-a-hacker.html
* Windows Subsystem for Linux- https://msdn.microsoft.com/en-us/commandline/wsl/install_guide
* Jekyll Web Site- https://jekyllrb.com/
* Git Basics- https://git-scm.com/book/en/v1/Getting-Started-Git-Basics

* Poole Web Site- http://getpoole.com/
* Creating and Hosting a Personal Site on GitHub- http://jmcglone.com/guides/github-pages/
* Run Jekyll on Windows- http://jekyll-windows.juthilo.com/
* Git Jekyll Cheatsheet- http://slhogle.github.io/2017/gitjekyll-cheatsheet/
* Markdown Cheat Sheet- http://packetlife.net/media/library/16/Markdown.pdf
* Other platforms, other walk throughs-
* http://joshualande.com/jekyll-github-pages-poole
* http://mlampros.github.io/2016/01/31/first-blog-post/
* https://www.smashingmagazine.com/2014/08/build-blog-jekyll-github-pages/

### Revision History
* 02/18/2019- Writing begins
* Content review-
* Style Guide review-
* Published-
