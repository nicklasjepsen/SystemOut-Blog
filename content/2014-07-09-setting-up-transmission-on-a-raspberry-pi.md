---
title: "How To Host a Transmission Daemon part 2"
author: "Nicklas Møller Jepsen"
date: "2014-07-09"
url: "/2014/07/09/setting-up-transmission-on-a-raspberry-pi/"
comments: "true"
categories:
  - off-topic
tags:
  - rasperry pi
  - torrent
  - transmission
  - self hosting
  - raspbian
  - learning by doing
series:
---
##  Setting Up Transmission on a Raspberry Pi

Before you start, please read why **I'm not using the Pi for Transmission**: [http://systemout.net/2014/07/09/how-to-host-a-transmission-daemon/](http://systemout.net/2014/07/09/how-to-host-a-transmission-daemon/)

The setup: A Raspberry Pi with Raspbian, a NAS with a share for the downloads
As you probably are well aware of the Pi OS is installed on a SD card. The Pi needs an OS and for that I used Raspbian.
Normally I am a Windows user and there fore I used a tool called Win32 Disk Imager to prepare the SD card with Raspbian. It is easy to use and can be downloaded here: [http://sourceforge.net/projects/win32diskimager/](http://sourceforge.net/projects/win32diskimager/) <!--more-->
You just give it the Raspbian IMG file that you can download (as a torrent, of course), from here: [http://downloads.raspberrypi.org/raspbian_latest.torrent](http://downloads.raspberrypi.org/raspbian_latest.torrent)

Once this is done, insert the card in to Pi, boot the device and follow the on screen instructions to finish configuring Raspbian. There’s nothing special to notice about the configuration with regards to using it with Transmission.

###  Mounting the share 
Because we want Transmission to download files to a network share, we need to mount this first.
There are many ways to mount a share, but I used quite a lot of time to find the most conveinient for my setup, because I needed the share to be mounted on reboot, there where some issues regarding the permissions and also some troubles with the security. But after all of that were tried and tested, I ended up with the following commands in terminal/putty/whatever:
(Open the fstab file, the place where mount points can be specified)

    sudo nano /etc/fstab

(Insert line at buttom)

    //<ip of NAS>/<name of share> /mnt/shares/downloads cifs username=<username on NAS>,password=<password on NAS>,sec=ntlm,uid=1000,gid=1000 0 0
Press Ctrl+X Y Enter

Okay, to sum up: We inserted a line in the fstab file telling that we want to mount a share on <ip of NAS>/<name of share> in location /mnt/shares/downloads. Further more we are using the cifs protocol and provide a username and password **used to log on to the NAS**. We need to define NTLM as the security we are using because of something with the cifs protocol (can’t remember were I read about it, but it works!). Last we are mounting with user 1000 (pi) as owners.

After that we save the fstab file by pressing Ctrl+X answering Y to save and Enter to overwrite.
Now we need to create the mount folder:

    sudo mkdir /mnt/shares/downloads
###  Installing Transmission 
First some basics:

    sudo apt-get update
	sudo apt-get upgrade    
Then install Transmission:

    sudo apt-get install transmission-daemon

Stop the Transmission Daemon, we need to do some configuration:

    sudo service transmission-daemon stop 
Then modify the default transmission user, the daemon-transmission:

    sudo usermod -a -G debian-transmission pi
And then let the user access the download folder:

    chgrp debian-transmission /mnt/shares/downloads
	chmod 770 /mnt/shares/downloads
Now we need to enable the Pi user (our user) to have ownership of Transmission files and add the Pi user to the debian-transmission group:

    sudo adduser pi debian-transmission 
	sudo nano /etc/init.d/transmission-daemon 
	sudo chown pi -R /var/lib/transmission-daemon/info/ 
	sudo chown pi -R /etc/transmission-daemon/settings.json 
	sudo chown pi -R /mnt/shares/downloads
Now before we start the Transmission Daemon, we need to change its configuration, to allow remote access and set the username and password. This is done by modifying the settings.json and we do this in nano:

	sudo nano /etc/transmission-daemon/settings.json
There are plenty of settings to play around with, but you probably want to change the following:

	“incomplete-dir-enabled”: false,
	“download-dir”: “/mnt/shares/downloads”,
	“rpc-password”: “<yourpassword>”,
	“rpc-username”: “<yourusername>”,
	“rpc-whitelist-enabled”: false,
Only set the whitelist setting, if you want to connect from other clients then the Pi you are installing on.
Save the file; Ctrl+X Y and Enter.

Now we can start Transmission:

	sudo service transmission-daemon start