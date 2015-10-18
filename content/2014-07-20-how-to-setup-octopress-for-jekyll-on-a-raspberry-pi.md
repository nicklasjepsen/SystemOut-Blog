---
title: "How To Setup Octopress for Jekyll on a Raspberry Pi"
author: "Nicklas Møller Jepsen"
date: "2014-07-20"
url: "/2014/07/20/how-to-setup-octopress-for-jekyll-on-a-raspberry-pi/"
comments: "true"
categories:
  - web development
tags:
  - raspberry pi
  - octopress
  - jekyll
  - linux
  - web hosting
  - raspbian
  - nginx
series:
---
### Introduction
There are several things we need to take care of to make a successful installation of Octopress on the Pi:

- Prepare the Raspbian for web hosting
- Install NGINX
- Install Git
- Install Curl
- Install RVM (Ruby Version Manager) and Ruby
- Finally install Octopress

But don't worry - all of the above is quite straight forward, although the Ruby installation might take some time.<!--more-->

#### Preparing Raspbian
First of you need a running Raspbian installation. The easiest way to get this, is by downloading it from [http://www.raspberrypi.org/downloads/](http://www.raspberrypi.org/downloads/).
Follow the official instructions to get up and running.

This tutorial will use the terminal/SSH from a remote computer and therefore we need to make sure we can reach it over the network.
To find the Pi's current IP write this:

	ifconfig
In my case I got a eth0 inet addr:192.168.X.YYY
When using the Pi as a web server it is a good idea to give it a static IP address. I have an DHCP server where this is handled, but if you want to set up a static IP on the Pi, here is how it is done: [https://wiki.debian.org/NetworkConfiguration#Configuring_the_interface_manually](https://wiki.debian.org/NetworkConfiguration#Configuring_the_interface_manually)

Now we can disconnect the screen and go back to our main desktop. I use Windows and therefore Putty ([http://the.earth.li/~sgtatham/putty/latest/x86/putty.exe](http://the.earth.li/~sgtatham/putty/latest/x86/putty.exe)) to connect to Pi.

I also like to give my computers a decent name (Raspbian sets the hostname to raspberrypi).
To change the hostname do the following:

	sudo namo /etc/hostname
Change the name to whatever you like and hit Ctrl+X, answer Y and the Enter to overwrite the file.
Now we need to update the hosts file:

	sudo nano /etc/hosts
Replace raspberrypi with the new hostname you provided in the previous step and hit Ctrl+X, answer Y and the Enter to overwrite the file.
Reboot the Pi:

	sudo reboot
For further info on changing the hostname of see: [https://wiki.debian.org/HowTo/ChangeHostname](https://wiki.debian.org/HowTo/ChangeHostname)

Now that Raspbian is installed and we have access to the Pi from SSH/Putty we need to make sure it is up to date by first collecting available updates and then upgrade/install them:

	sudo apt-get install update
	sudo apt-get install upgrade

### Install NGINX, Curl, Git and Ruby
First things first:

	sudo apt-get install nginx git curl

With my Raspbian install I actually only needed to install NGINX, but just run the above to be sure to have what wee need, no damage done.

No RVM and Ruby:

	\curl -L https://get.rvm.io | bash -s stable --ruby
...to get the latest stable RVM.
This process takes some time, on my Pi it took about **1 and  a half hour**, so go watch some Simpsons. :)

To start using RVM you need to run the command the installer tells you to or simply reconnect to the terminal.

According to the Octopress docs, [http://octopress.org/docs/setup/rvm/](http://octopress.org/docs/setup/rvm/), we need Ruby version 1.9.3.
Therefore we do the following:

	rvm install 1.9.3

This command also took an hour or so on my Pi, but the rest should be easyly done.

	rvm use 1.9.3
	rvm rubygems latest

And now check the the correct Ruby version is used:

	ruby --version

It should display 1.9.3-something.

### Installing Octopress
To install Octopress from Git run the following:

	git clone git://github.com/imathis/octopress.git octopress
...where the last option 'octopress' is the name of the site/directory for your Octopress installation. I name mine according to the site I'm going to host.

Now we need to install Octopress' dependencies.
Browse to the name of the directory which you specified in the git command, like so:

	cd octopress
and run

	gem install bundler
and lastly

	bundle install

To install the default Octopress theme:

	rake install

### Setting up NGINX
NGINX has a folder where it stores configuration for different sites.
We use nano to create a new configuration file for our site:

	sudo nano /etc/nginx/sites-available/DOMAIN

And then copy the following into the new file, remeber to change IPOFTHEPI and DOMAIN with your values. Also, if you gave another name for your Octopress installation in previeus steps, change the path in the location accordingly:

	server {
        listen IPOFTHEPI:80;
        server_name DOMAIN;

            location / {
                root /home/pi/octopress/public;
                index index.html index.htm;
                access_log /home/pi/octopress/source/YOURDOMAIN.access.log;
                error_log /home/pi/octopress/source/YOURDOMAIN.error.log;
            }
        }

Save the new file.

Create a link from the created file to the NGINX enabled sites directory:

	cd /etc/nginx/sites-enabled
	ln -s /etc/nginx/sites-available/DOMAIN /etc/nginx/sites-enabled/DOMAIN
	ls

Restart NGINX:

	/etc/init.d/nginx restart

And that's it!

Now all theres left to be done, is setting up your blog/site by following the guidelines on the official Octopress web site: [http://octopress.org/](http://octopress.org/)

If you want to ease the generation and deployment process, you can build the blog on your windows machine and deploy it to the Raspberry Pi in a single click. Learn how to set this up by reading my guide here: [http://systemout.net/2014/12/18/how-to-build-and-deploy-octopress-with-a-single-click-p1/](http://systemout.net/2014/12/18/how-to-build-and-deploy-octopress-with-a-single-click-p1/)