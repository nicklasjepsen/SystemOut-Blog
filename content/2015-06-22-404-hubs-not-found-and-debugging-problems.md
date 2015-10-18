---
title: "404 Hubs Not Found And Debugging Problems"
author: "Nicklas Møller Jepsen"
date: "2015-06-22"
url: "/2014/07/17/404-hubs-not-found-and-debugging-problems/"
comments: "true"
categories:
  - programming
  - signalr
tags:
  - c-sharp
  - iis
  - deployment
  - chat
  - tips and tricks
series:
---
## Introduction
When working with SignalR I sometimes stumble upon some irritating problems that mostly seems related to Visual Studio and maybe also related to the SignalR framework being relatively new.

In this post I have assembled some of the errors I have been facing while developing with SignalR.<!--more-->

### 404 Hub(s) not found
This error seems to have a different solution whether you have deployed your app to IIS or you are debugging.

##### Debugging
I have experienced a lot of difficulties while debugging/running my SignalR apps in Visual Studio 2013. Sometimes, when I reopen a solution that previously “just worked”, it all of the sudden don’t anymore, meaning that the ~/signalr/hubs gives a 404 on localhost or the client can not be found. A fix for this seems to be the following:

1. Close Visual Studio
2. Make sure IIS Express is shutdown also (look for the IIS Express tray icon)
3. Start Visual Studio

If the above doesn't work, sometimes going out for a cup of coffee and returning does :)

##### Deployed on IIS
This is most likely related to the hub reference and how you are hosting the SignalR site on the server. I have written a post about the case, which you can find [right here](http://systemout.net/2014/07/15/signalr-hub-reference-done-right/).

### Troubleshooting
Remember that you can enable detailed errors when trying to troubleshoot SignalR related problems. Just add this to your OWIN Startup.cs:

{{< highlight csharp >}}
var hubConfiguration = new HubConfiguration
{
	EnableDetailedErrors = true,
};
app.MapSignalR("/~/signalr", hubConfiguration);
{{< /highlight >}}

Leave a comment if you have any other problems, and I'll be glad to try and help out!