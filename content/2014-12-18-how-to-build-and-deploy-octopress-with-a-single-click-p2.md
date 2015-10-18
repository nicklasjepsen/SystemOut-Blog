---
title: "How to build and deploy octopress with a single click, part 2 "
author: "Nicklas Møller Jepsen"
comments: "true"
date: "2014-12-18"
url: "/2014/12/18/how-to-build-and-deploy-octopress-with-a-single-click-p2"
comments: "true"
categories:
  - web development
tags:
  - blogging
  - winscp
  - ftp
  - build
  - deploy
  - rake
  - ruby
  - git
  - script
  - raspberry pi
  - octopress
series:
---
## Introduction
This post is a continuation of the post about generating an Octopress site on a Windows machine and deploying it to a web server/Raspberry Pi.<!--more-->
You can find the post about setting up your Windows dev box to generate the Octopress site [right here](http://systemout.net/2014/12/18/how-to-build-and-deploy-octopress-with-a-single-click-p1/)!

## Steps
1. [Donwload and install WinSCP](#winscp)
2. [Enable FTP on the Raspberry Pi/web server](#ftp)
2. [Create WinSCP script](#winscpscript)
3. [Create build and deployment script](#deploy)
4. [Run!](#run)


### Download and install WinSCP<a name="winscp"></a>
You can grab the latest version of WinSCP from [here](http://winscp.net/eng/download.php).
Just follow the installation instructions and you are done!

### Enable FTP on the Raspberry Pi/web server<a name="ftp"></a>
I use [ProFTPD](http://www.proftpd.org/) as my FTP server on the Raspberry Pi running Raspbian. On Raspbian/Debian it can be installed like this:
{{< highlight  bat >}}
	apt-get install proftpd
{{< /highlight >}}
Choose the standalone option when prompted.
After installation is completed, the FTP server should be running. 
If you for some reason need to, you can stop/start the FTP server as follows:
{{< highlight  bat >}}
	/etc/init.d/proftpd stop
	/etc/init.d/proftpd start
{{< /highlight >}}
Now that's up and running, we need to create a WinSCP script to upload the generated files.

### Create WinSCP script<a name="winscpscript"></a>
Besides being a gui tool for file handling, WinSCP comes with a command line interface. Refer to the documentation [here](http://winscp.net/eng/docs/scriptcommand_synchronize) if you need further information.



Create a new text file, name it winscpscript.scp and place it in octopress\scripts (create the scripts folder, if it doesn't exist) and insert the following code:
{{< highlight  bat >}}
	option batch abort
	option confirm off
	open username:password@ipOrHostOfWebserver
	cd /srv/www/octopress
	lcd C:\octopress\public\
	synchronize remote -delete
	exit
{{< /highlight >}}
In this script, the following happens:
The first 2 lines ensures that prompts are answered negativly and that confirmations are turned of.

The open command connects to the FTP server using the provided username/password on the hostname or ip address of you web server.

After that we change the remote directory (the directory on the server), using the cd command, to a suitable place to store our web site.

Then we use the lcd command to change the local directory to the location on the Windows computer where Octopress is installed/generated. Remember that the output of the generation is in the 'public' directory.

**Now the magic!**

Using the syncrhonize command we can sync the files in the local directory with the remote directory. The remote option tells WinSCP that it is the remote directory that shall be updated with changes from the local directory. By adding the -delete option, we tell WinSCP that the synchronization should make sure that locally deleted files also gets deleted from the remote directory.

Lastly we simply exit the WinSCP CLI and returning control to the caller.

### Create build and deployment script<a name="deploy"></a>
Now we can create the build script that will enable us to build/generate and publish our Octopress site with one single click.

Create new text file, name it BuildOctopress.bat and save it in octopress\scripts with the following content:
{{< highlight  bat >}}
	cd "C:\octopress\"
	call rake generate || true
	cd "C:\Program Files (x86)\WinSCP\"
	winscp /script=C:\octopress\scripts\winscpscript.scp
{{< /highlight >}}
Now here is what happens in the script:

First we change working directory to the location of out octopress site.

Then we generate the site using rake generate. The call function is handy, because we're calling another batch job.

Now cd to the WinSCP directory.

Run WinSCP and pass the script we created in the previous step.

Save the file and continue to the next step.

### Run<a name="run"></a>
After all the above is done, we are now able to build and deploy out Octopress site by the single click of a mouse. Simply run the BuildOctopress.bat file and wait for it to finish.


And that's it. Please feel free to leave any comments and/or questions.
