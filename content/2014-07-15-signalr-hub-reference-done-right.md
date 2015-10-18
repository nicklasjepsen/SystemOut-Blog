---
title: "SignalR Hub Reference - done right!"
author: "Nicklas Møller Jepsen"
date: "2014-07-15"
url: "/2014/07/15/signalr-hub-reference-done-right/"
comments: "true"
categories:
  - web development
  - signalr
  - programming
tags:
  - c-sharp
  - iis
  - signalr
  - owin
  - guide
series:
---
UPDATE: I have now written a complete SignalR tutorial, see this post: http://systemout.net/2014/07/17/developing-and-deploying-a-signalr-chat-web-app-to-iis/

SignalR is nice. No doubt about that. But it is very much immature, not the framework, more the tutorials, guidelines, etc. 

I have been strugling a bit with the Javascript/client setup of the hub connection - especially when deploying to an IIS 8 in production, where all the magic stopped to happen. There is not much to gain from WebSockets, if the connection can't be opened because the scripts won't load.<!--more-->

Any way, in an ASP.NET Web Application using SignalR with a generated hub proxy you need the following things to be able to make the connection:
OWIN Startup.cs (Add > New Item > Owin Startup):
	
	using Microsoft.AspNet.SignalR;
	using Microsoft.Owin;
	using Owin;
	
	[assembly: OwinStartup(typeof(SystemOut.Gateway.WebApp.Startup))]

	namespace SystemOut.Gateway.WebApp
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

**NOTICE** the "/~/signalr - this is needed, if you, as I, are running the site in your production environment as a sub directory for the main site.

Because of this, you need to reference the generated SignalR proxy a little different than most samples I came across do:

  	<script src="~/signalr/js"></script>

You also need to create the hub connection a little bit different, in your scripts:

	$.connection.hub.start("~/signalr")

That should do it!

Remember, you can always enable logging to the browsers console (F12) by adding this line:
	
	$.connection.hub.logging = true;
just before:

	$.connection.hub.start("~/signalr")

#### The trouble
Before settings things up this way, I received a "Unable to get property 'client' of undefined or null reference" in the IE console when loading the page. In that case, the "client" referenced was the SignalR client in the Hub.
I investigated it further by looking in Fiddler and in Chrome. In the console in Chrome I got "http://myip/signalr/js 404 (Not Found)" which lead me to come up with the above solution, becuase I only had the problem when deploying the application to IIS - not when I ran it locally from Visual Studio 2013.

As an end note; it makes no difference for me whether I use:
	
	~/signalr/js 
or

	~/signalr/hubs

That's it for now. I'm working on a full tutorial to developing AND deploying an ASP.NET Web Application with SignalR, so check back soon! :)

UPDATE: I have now written a complete SignalR tutorial, see this post: http://systemout.net/2014/07/17/developing-and-deploying-a-signalr-chat-web-app-to-iis/
