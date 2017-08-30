---
title: "Developing and Deploying a SignalR Chat Web App to IIS"
author: "Nicklas Møller Jepsen"
date: "2014-07-17"
updated: "2015-06-10"
url: "/2014/07/17/developing-and-deploying-a-signalr-chat-web-app-to-iis/"
comments: "true"
categories:
  - web development
  - signalr
  - programming
tags:
  - c-sharp
  - iis
  - owin
  - tutorial
  - deployment
  - chat
series:
---
## Introduction

Download the complete solution [here](https://drive.google.com/file/d/0B_b7_Dquxu0KWjlCWVJyVUlZbGM/view?usp=sharing)

In this tutorial I'm going to show you how you with ease can develop and deploy you very own SignalR Chat App!

The SignalR framework is great for bidirectional communication becauses it uses WebSockets. SignalR handles all the connections and you just need to focuse on implementing your business logic.
**But**, I have found, that it can be a bit of a fuzz to setup, and to get a connection to the SignalR Hub (which I'll explain in details in a bit), and therefore I wrote this tutorial.<!--more-->

*UPDATE*: I have posted [a new tutorial](http://systemout.net/2015/06/10/developing-a-who-is-on-chat-web-app-using-signalr-and-wpf/) showing how it is possible to let the clients see who is online. That post builds on this tutorial, so check it out after you have read this one!


### The Setup
We are going to need the following:

- A HTML page
- A javaScript
- A MessageHub (C\#)
- A Startup file (also C\#)

All of the above wrapped in a ASP.NET Web Application solution.

Before we begin, you can take a look at the product right here:
[http://systemout.net:56677/ChatWebApp/Messenger.html?username=User1](http://systemout.net:56677/ChatWebApp/Messenger.html?username=User1)

## Implementing the Solution
An important note: It is critical that you create a **ASP.NET Web Application** and **NOT** a ASP.NET Web Site. Maybe you know that all future web apps should be created as such, as stated by Microsoft, but I didn't and learned it the hard way (meaning my SignalR app didn't work and it took some hours figuring out why not...).

But let's get started; fire up Visual Studio, and select File > New > **New Project**
![CPU usage](http://systemout.net/images/NewAspNewWebApplication.png)

Name it ChatWebApp.

Create and empty project:
![CPU usage](http://systemout.net/images/EmptyProject.png)

#### SignalR NuGet packages 
There are several ways to start of creating a SignalR enabled project, and you can add a new Hub or a OWIN Startup class directly from Visual Studio's context menues, but I found that I didn't get the lates stable SignalR version when doing so.
Therefore, we are going to add the NuGet packages needed manually:
Right click your new project > Manage NuGet Packages
Click Install to Microsoft ASP.NET SignalR:
![CPU usage](http://systemout.net/images/SignalRNuget.png)
It will also ask you to install various other packages needed, but the hole process is automated, so no trouble there.

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- links1 -->
<ins class="adsbygoogle"
     style="display:inline-block;width:728px;height:15px"
     data-ad-client="ca-pub-5807169669170468"
     data-ad-slot="6444119354"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

#### OWIN Startup
To do the wire up/mapping of SignalR wee use OWIN.
Simply right click your project, and select Add. Search for OWIN and add a new Startup.cs file.
Replace the content of the file with this:   

{{< highlight  csharp >}}

	using System;
	using System.Threading.Tasks;
	using Microsoft.AspNet.SignalR;
	using Microsoft.Owin;
	using Owin;

	[assembly: OwinStartup(typeof(ChatWebApp.Startup))]

	namespace ChatWebApp
	{
    	public class Startup
    	{
        	public void Configuration(IAppBuilder app)
        	{
            	// For more information on how to configure your application, visit http://go.microsoft.com/fwlink/?LinkID=316888
            	app.MapSignalR("/~/signalr", new HubConfiguration());
        	}
    	}
	}
{{< /highlight >}}

The important things here are of course the assembly annotation where we declare that the type ChatWebApp.Startup (this class) should be used by OWIN.
The next thing to note is the MapSignalR method call. By default, VS just uses the MapSignalR method without any parameters, and this works well - when developing on running locallyt, but I have got it to work in a production IIS. Therefore the manually mapping path used for SignalR.

#### The Hub
Now we can add a Hub Class from the conect menu in Visual Studio, because we already added the references manually:
Right click your project > Add > Signal Hub Class (v2). If it isn't there just select Add New Item and search for it.
Name it MessageHub.cs

Replace the content of the file with this:   

{{< highlight  csharp >}}

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Web;
    using Microsoft.AspNet.SignalR;

    namespace ChatWebApp
    {
        public class MessageHub : Hub
        {
            public void Login(string username)
            {
                // Notify all clients that a user is logged in
                Clients.All.userLoggedIn(username);
            }

            public void SendMessage(string sender, string message)
            {
                // Display the new message by calling displayMessage on all connected clients
                Clients.All.displayMessage(sender, message);
            }
        }
    }
{{< /highlight >}}

Here we are adding a method used to "log in" to our Chat App and a method to send messages.
It should be pretty self explanotory but what we are doing is telling all clients that either a usre has logged in or to display a message. 
The userLoggedIn(username) and displayMessage(sender, message) methods are implemented in JavaScript in just a moment. 

For further information about the workings of the Hub, please go to:
[http://www.asp.net/signalr/overview/signalr-20/hubs-api/hubs-api-guide-server](http://www.asp.net/signalr/overview/signalr-20/hubs-api/hubs-api-guide-server)

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- ResponsiveHeader -->
<ins class="adsbygoogle"
     style="display:block"
     data-ad-client="ca-pub-5807169669170468"
     data-ad-slot="5652122954"
     data-ad-format="auto"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

#### The Messenger JavaScript
Now we have all the C# code we need and can go on to the client side JavaScript.
Add a new JavaScript file, Add > Add New Item > JavaScript and name it MessengerScript.js

Replace the content of the file with this:   

{{< highlight  js >}}

	$(function () {
	    var username = getUrlVars()["username"];
	
	    // Enable logging for development purpose
	    $.connection.hub.logging = true;
	    // Declare a proxy to reference the hub.
	    var messageHub = $.connection.messageHub;
	    // Create a function that the hub can call to broadcast messages.
	    messageHub.client.displayMessage = function (sender, message) {
	        // Wrap the sender and the message in HTML
	        var senderDiv = $('<div />').text(sender).html();
	        var messageDiv = $('<div />').text(message).html();
	        // Add the message to the page.
	        $('#messagelist').append('<li><strong>' + senderDiv
	            + '</strong>:&nbsp;&nbsp;' + messageDiv + '</li>');
	    };
	    
	    $.connection.hub.start("~/signalr").done(function () {
	       
	        messageHub.server.login(username);
	
	        $('#sendmessage').click(function () {
	            var msg = $("#message").val();
	            messageHub.server.sendMessage(username, msg);
	            // Clear text box and reset focus for next comment.
	            $('#message').val('').focus();
	        });
	    });
	});
	
	function getUrlVars() {
	    var vars = [], hash;
	    var hashes = window.location.href.slice(window.location.href.indexOf('?') + 1).split('&');
	    for (var i = 0; i < hashes.length; i++) {
	        hash = hashes[i].split('=');
	        vars.push(hash[0]);
	        vars[hash[0]] = hash[1];
	    }
	    return vars;
	}
	
{{< /highlight >}}

Now a brief note; this is actually the first JavaScript code I have written, so there might be better ways to implement the above, but it works, and the purpose of this tutorial is to show how to use SignalR - not how JavaScript should be written, so bear with me, please :)

The first part gets the username from a query parameter by calling the last function in the script the getUrlVars (copy pasted somewhere from StackOverflow, can't remember where I found it thou).
Then we enable logging in the SignalR connection, get a reference to the MessageHub class that we just implemented, and now some of the "magic": **implementing the displayMessage method** as we are going to invoke from our MessageHub. The displayMessage is basically wrapping the sender and message in some HTML and added a li to a ul declared in a HTML file we will create in the next step.

We need to declare the displayMessage method before connecting to the hub so now that this is done we start a connection to the hub and gives it the path as we used in the Startup.cs. Then, when the connection is done we are calling login on the hub and last but not least **implementing the sendMessage method** which gets the message from a input box (which we also will implement in a HTML file in a moment) and then we call the SendMessage method on the hub.

#### The HTML Page
Next we add the HTML page which the user sees. Add > New Item > HTML Page. Name it Messenger.html
Replace the content of the file with this:  

{{< highlight  xml >}}

    <!DOCTYPE html>
	<html xmlns="http://www.w3.org/1999/xhtml">
	<head>
	    <title>ChatWebApp</title>
	</head>
	    <body>
	        <script src="Scripts/jquery-1.6.4.js"></script>
	        <script src="Scripts/jquery.signalR-2.1.0.js"></script>
	        <script src="~/signalr/hub"></script>
	        <script src="Scripts/MessengerScript.js"></script>
	        <input id="message" type="text"/>
	        <input id="sendmessage" type="button" value="Send"/>
	        <ul id="messagelist">
	            
	        </ul>
	    </body>
	</html>
{{< /highlight >}}

Now the most important path here, are:

{{< highlight  xml >}}

	<script src="~/signalr/hub"></script>
{{< /highlight >}}

Visual Studio will complain about this, because we have no script in the path, but this is because the script is generated by SignalR on runtime and we are just telling where it can be found.
Besides we have the elements used to interact with our system.

With this in place, you can run your new application and test it in your browser, before continuing to the next part, where we will deploy to IIS.

#### A note on debugging in VS
I have experienced a lot of difficulties while debugging/running my SignalR apps in Visual Studio 2013. Sometimes, when I reopen a solution that previously "just worked", it all of the sudden don't anymore, meaning that the ~/signalr/js gives a 404 on localhost or the client can not be found.
A fix for this seems to be restarting VS, restarting the IIS Express by right clicking on the tray icon:
![](http://systemout.net/images/IISExpress.png)
...or simply just hitting my screen, going for some coffee and when returning, it magically just works.

### Deploying the SignalR App to IIS
We use Visual Studio to publish the site. Right-click your project, select Publish
In the dialog click the Custom option as publish target and give it a name.
Select File System in the next drop down and enter a location for the published files.
In settings you can choose to precompile your source files, if you want to, just expand the File Publish Options and check the according box.

Click Publish.

Now we that we have our files ready, we need to make the IIS ready for hosting our App.

On your server, create the folder C:\wwwroot\ChatWebApp (or anywhere else you like).
Copy the contents from the Publish folder (on your dev pc) where Visual Studio published the files to into C:\wwwroot\ChatWebApp on IIS.
On your server open IIS Manager, expand Default Website. If your new folder is in the tree view right click and select Refresh.
Select the ChatWebApp folder, right click and select Convert to Application.

Now on your server go to http://localhost/ChatWebApp/Messenger.html?username=MySuperDupperUser and watch the magic!


You can download the complete solution right here: [http://1drv.ms/1wB5IWF](http://1drv.ms/1wB5IWF) 

If you have in problems, or comments, let me know.

*UPDATE*: I have posted [a new tutorial](http://systemout.net/2015/06/10/developing-a-who-is-on-chat-web-app-using-signalr-and-wpf/) showing how it is possible to let the clients see who is online. That post builds on this tutorial, so check it out after you have read this one!
