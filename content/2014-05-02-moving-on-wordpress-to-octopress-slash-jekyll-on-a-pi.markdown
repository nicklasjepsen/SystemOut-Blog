---
title: "'Moving on – WordPress to Octopress/Jekyll on a Pi'"
date: "2014-05-02"
url: "/2014/05/02/moving-on-wordpress-to-octopress-slash-jekyll-on-a-pi/"
comments: "true"
categories:
  - web development
tags:
  - blogging
  - jekyll
  - octopress
  - raspberry pi
  - migration
  - web hosting
  - ruby
  - nginx
series:
---
This 1 post is part of a series of posts.

- Summary (this post)
-	The Pi: http://systemout.net/2014/07/20/how-to-setup-octopress-for-jekyll-on-a-raspberry-pi/
-	The Blog
-	The Rest (including the network)

###  Introduction
The look ’n feel of the blog has changed, as you may have noticed. 
The reason for this is simple; slow performance, increasing cost to hosting and pure curiosity, got me into migrating the existing WordPress solution into a static Octopress on Jekyll solution. Oh, and the new solution is hosted on a Raspberry Pi, <a href="http://swag.raspberrypi.org/collections/frontpage/products/raspberry-pi-model-b" target="_blank">model B</a>.

The result is amazing – the average response time is down around 400 MS after little over a week on the pie. Before it was about 1,5-2 secs. 
In this post I’ll explain how the migration is done and which challenges I faced. I will not go into details, but I will provide you with the links to some great articles I used along the way.
<!--more-->
###  Summary
First a brief summary of the steps required.
The Pi:

* Install Raspbian
* Setup static IP
* Install NGINX
* Install RVM
* Install Ruby
* Install Octopress
* Configuration of NGINX/Octopress

The Blog:

* Download existing post, comments, pages, etc
* Convert to Octopress/Jekyll format
* Upload to Pi
* Build the new Blog

The Network:

* DNS
* NAT on home network
* Limit torrents!

In the next post, I will dig into the details in configuring the Pi, and that post is available right here: http://systemout.net/2014/07/20/how-to-setup-octopress-for-jekyll-on-a-raspberry-pi/
