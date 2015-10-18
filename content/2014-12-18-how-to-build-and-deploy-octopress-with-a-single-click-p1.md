---
title: "How to build and deploy octopress with a single click, part 1"
author: "Nicklas Møller Jepsen"
date: "2014-12-18"
updated: "2015-08-09"
url: "/2014/12/18/how-to-build-and-deploy-octopress-with-a-single-click-p1"
comments: "true"
categories:
  - web development
tags:
  - raspberry pi
  - octopress
  - ruby
  - git
  - build
  - deploy
  - rake
series:
---
**UPDATE**
It is no longer required to use Ruby 1.9.3 with Octopress so the post has been updated accordingly.
Also, installation instructions for Python is added because this is required for the build process to work.

## Introduction
In this guide I will guide you to how you can build and deploy your Octopress site with a single mouse click. That is; building Octopress on your Windows computer and deploying the generated site to your web server, for example a Raspberry Pi!
To accomplish this, we will be using Ruby, rake and WinSCP and deploying it using FTP.<!--more-->

This blog, [Systemout.net](http://systemout.net "SystemOut.net"), started out just being generated on the web server/the Pi, but the Pi is not exactly made for CPU intensive work and therefore I got a little tied of waiting for even the smallest changes made to the blog to be generated. 
I'm sure you know this, but Octopress needs to regenerate the entire site, primarily to keep all links inbetween the various post valid, even if something is changed. This is the disadvantage of having a static site as your blog, BUT once the generation/build/deployment process is over, you can enjoy the benefits of a static blog: Speed, simplicity, etc.

## Steps
There are a few steps required before you can use your Windows box to build and host Octopress on it.
But don't worry: If you follow this guide, it will be a peace of cake!
The steps required are:

1. [Install Ruby and the Ruby SDK](#ruby)
3. [Install Tortoise Git](#tortoise)
4. [Clone Octopress](#clone)
5. [Generate your Octopress site and preview](generate)
6. [Deploy](#deploy)

### Installing Ruby and the Ruby SDK on Windows<a name="ruby"></a>

You can find the various Ruby downloads [here](http://rubyinstaller.org/downloads/).

[And here is a direct download link for Ruby 2.2.2](http://dl.bintray.com/oneclick/rubyinstaller/rubyinstaller-2.2.2.exe).

For Ruby version 2.2.2 we need the mingw64-32-4.7.2 of the Ruby SDK. This can be found [here](http://dl.bintray.com/oneclick/rubyinstaller/DevKit-mingw64-32-4.7.2-20130224-1151-sfx.exe).

We need to make some manual changes for the Ruby SDK to function properbly. Open a command prompt and cd to the location where you extracted the Ruby SDK, let's say it is in C:\RubySdk and run the following 3 commands:
{{< highlight  bat >}}
    C:\RubySdk>devkitvars.bat
    C:\RubySdk>ruby dk.rb init
    C:\RubySdk>ruby dk.rb install
{{< /highlight >}}
Lastly we can test to see that the correct version of Ruby is installed by running:
{{< highlight  bat >}}
	C:\>ruby --version
{{< /highlight >}}
which should give us something like: 
{{< highlight  bat >}}
	ruby 1.9.3p551 (2014-11-13) [i386-mingw32]
{{< /highlight >}}
...and that's it for the Ruby installation!

### Install Python
Pretty simple: Go to [the official site](https://www.python.org/downloads/windows/) to get the latest installer and install it.

### Install Tortoise Git<a name="tortoise"></a>
This step is pretty straight forward; simply download and install the Tortoise Git client from [here](https://code.google.com/p/tortoisegit/wiki/Download)!

### Clone Octopress<a name="clone"></a>
Now we're setup and ready the start with the fun stuf!

Open a command prompt, cd to the directory where you want your Octopress cloned into and run:
{{< highlight  bat >}}
    git clone git://github.com/imathis/octopress.git octopress
{{< /highlight >}}
Now cd to the octopress dir and install some gems:
{{< highlight  bat >}}
	cd octopress
	gem install bundler
	bundle install
{{< /highlight >}}
**Note:** If you get an error like:

{{< highlight  bat >}}
	Gem::RemoteFetcher::FetchError: SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B:certificate verify failed (https://rubygems.org/gems/listen-2.8.4.gem)
{{< /highlight >}}
Then you need to change the URL of the rubygems location. This is done in the Gemfile. The Gemfile is located in the Octopress directory as you just cloned from Git and **after** correction it should look like:

{{< highlight  ruby >}}
	source "http://rubygems.org"

	group :development do
  		gem 'rake', '~> 10.0'
  		gem 'jekyll', '~> 2.0'
  		gem 'octopress-hooks', '~> 2.2'
  		gem 'octopress-date-format', '~> 2.0'
  		gem 'jekyll-sitemap'
  		gem 'rdiscount', '~> 2.0'
  		gem 'RedCloth', '~> 4.2.9'
  		gem 'haml', '~> 4.0'
  		gem 'compass', '~> 0.12.2'
  		gem 'sass-globbing', '~> 1.0.0'
  		gem 'rubypants', '~> 0.2.0'
  		gem 'rb-fsevent', '~> 0.9'
  		gem 'stringex', '~> 1.4.0'
	end

	gem 'sinatra', '~> 1.4.2'
{{< /highlight >}}
Save the Gemfile and rerun the command
{{< highlight  bat >}}
	bundle install
{{< /highlight >}}
Finally we need to run this command:
{{< highlight  bat >}}
	rake install
{{< /highlight >}}
And that's it - now Ruby is installed, Octopress is cloned and ready to be generated!

### Generate your Octopress site and preview<a name="generate"></a>
Now we need to build our Octopress site using the generate command:
{{< highlight  bat >}}
	rake generate
{{< /highlight >}}
This should take no time and now we can start a web server for previewing the generated site:
{{< highlight  bat >}}
	rake preview
{{< /highlight >}}
Now browse to [localhost:4000](http://localhost:4000) - and you should see you new Octopress site! :)


### Deploying to you web server<a name="deploy"></a>

You could of course just keep you blog hosted on localhost:4000, but what's the fun in that?
No where - therefore I'm going to show you how to deploy the generated blog to you Raspberry Pi or other web server using a little bit of command line magic combined with WinSCP and FTP. There are some steps involved in this so I wrote another blog post of how to do that. You can find the post [right here](http://systemout.net/2014/12/18/how-to-build-and-deploy-octopress-with-a-single-click-p2/)!

And that's it! Feel free to leave any comments/ask questions in the comment fields below.

Thanks for reading :)