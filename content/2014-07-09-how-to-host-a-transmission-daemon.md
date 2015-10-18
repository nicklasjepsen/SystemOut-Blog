---
title: "How To Host a Transmission Daemon part 1"
author: "Nicklas Møller Jepsen"
date: "2014-07-09"
url: "/2014/07/09/how-to-host-a-transmission-daemon/"
comments: "true"
categories:
  - off-topic
tags:
  - torrent
  - transmission
  - self hosting
  - raspbian
  - windows server 2012 r2
  - learning by doing
series:
---
If you want to skip to the "How to install transmission daemon on the Pi, look here: [http://systemout.net/2014/07/09/setting-up-transmission-on-a-raspberry-pi/](http://systemout.net/2014/07/09/setting-up-transmission-on-a-raspberry-pi/)

##  First a disclaimer...

I started out wanting to host a Transmission Daemon on a Raspberry Pi for some simple reasons; I wanted a torrent client which was always on, accessible from all my devices, and easy to use. I had a Transmission Daemon running on my NAS, but I have decided to let my NAS do what is is best at: Serving as a storage device, and nothing more. To keep the cost down, and because I thought it would be exciting (which it was!), I decided to install Transmission on a Raspberry Pi.

**But then all the 'fun' began...**

Short story: I mounted a network share where the Pi shoud download files to, after long strugling whith permissions I found out that the CPU usage was extensive when writing directly from Transmission to the share. Or, I thought that was the issue.. <!--more-->
To optimize I mounted a USB drive to the Pi and used it as a placeholder for downloads in progress only to see that the CPU usage was still in the extreme zone.

That's why I decided to go back to my Hyper-V hosted Transmission Daemon.

For all of this not to be a total waste of time (oh btw, it isn't - I've learned a lot), I have decided to publish the procedure of how I did setup the Pi to host Transmission, how I mounted the share, how I formatted the external USB disk and how it all came up and running.
Furthermore I will describe how I installed Transmission on Windows Server 2012 r2 and how it is running now.

###  Why I skipped the Transmission Daemon on the Pi 
Look at the CPU usage Transmission:
![CPU usage](http://systemout.net/images/TopCpu.png)

And I did only manage to get download speeds of around 1-2 mb/s:
![Trasnmission UI](http://systemout.net/images/TransmissionWeb.png)

Now, have a look at the post about how to set the Transmission up on the Pi and please leave comments, if you find a better way, or even a way where you are able to get descent download speeds:
[http://systemout.net/2014/07/09/setting-up-transmission-on-a-raspberry-pi](http://systemout.net/2014/07/09/setting-up-transmission-on-a-raspberry-pi)




