---
title: "Backup to OneDrive or Other Cloud Storage Using a Simpel Script"
author: "Nicklas Møller Jepsen"
comments: "true"
date: "2014-12-29"
url: "/2014/12/29/backup-to-onedrive-or-other-cloud-storage-using-a-simpel-script"
comments: "true"
categories:
  - windows
tags:
  - backup
  - onedrive
  - cloud
  - script
series:
---

## Getting Started
I have a Office 365 Home/Family subscription that I use to get access to the latest version of the Office pack, get 1 TB (yes, tera byte) of included OneDrive storage and best of all; included in the subscription, is the possibility to add up to 4 other members of my family. That means that my girlfriend can get 1 TB online storage, without paying a dime, and for me that means I don't have to worry about helping her backup all of her photos and other stuff.<!--more-->

Now this is NOT a commecial for Microsoft/OneDrive, but the product they are offering are just amazing!
To backup your files to the cloud, as I will walk you through in this guide, you can use any of the existing cloud storage providers; Dropbox, OneDrive/SkyDrive, you name it.

As a developer I have a ton of files on my system that I would hate to loose. This includes all the source code for the countless amount of projects I have worked upon in my spare time.
All of these files I like to be able to control where they are physically stored on my hard drive, for instance I like to have a folder in C:\Dev where I store all my development projects.
Surely I have set up a local subversion server hosted on my NAS, but what if the house burns to the ground?
Therefore I needed to set up a backup of the subversion files on the NAS to be copied to OneDrive. 
Here's how it's done!

1. Create a BAT file
2. Create a Scheduled Task

And that's it!

### The BAT Script, ROBOCOPY
I'm going to use ROBOCOPY, simple because it's amazing and has some features that I need for things to work the way I intend.
Below is the command that we're going to run, in all its flavor. I will explain each part of the command afterwards.

{{< highlight shell-session >}}
robocopy X:\.subversion C:\Users\Nicklas\SkyDrive\Backup\MYNAS\.subversion /e /mir /log:C:\Users\Nicklas\SkyDrive\Backup\subversion_backup_log_%date:~-4,4%%date:~-10,2%%date:~-7,2%_%time:~0,2%-%time:~3,2%-%time:~6,2%.txt
{{< /highlight >}}

The syntax is as follows:

{{< highlight  shell-session >}}
robocopy [source] [destination] params 
{{< /highlight >}}

Source and destination should be obvious.
The parameters explained:
/e : Copies all subfolders
/mir : Will only copy newer/added files. Making the backup kind of incremental.
/log : The path to store the log file.

Speaking of the log file; I using the syntaxt above to get a timestamp to put in the file name of the log file - otherwise it will just get overridden each time the backup is running. Depending on your Windows locale, that syntaxt might be different.

### Scheduled Task
Open the the Task Scheduler, goto Run and enter:
{{< highlight bat >}}
Taskschd.msc
{{< /highlight >}}

Choose Create Task.
Give it a name and select the **Triggers tab** and then **New**. Select **Daily** and leave the rest as defaults.
Select the **Actions tab** and then **New**.
Enter the path to the BAT script created in the previous step (I have stored mine in my OneDrive backup folder for ease of access).
Then the last thing: Go to **Settings tab** click the **If the task fails, restart every** to make sure that the backup will be performed.

And we're done!

## Important
This wil only copy/replace new and existing files, it will not delete files no longer in the source.
A fix for this is to manually "resetting" the destination files by deleting them and running a full backup from scratch one in a while. 