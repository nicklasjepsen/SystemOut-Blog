---
title: "Calling SignalR Hub from a C# WCF Service"
author: "Nicklas Møller Jepsen"
date: "2014-07-25"
updated: "2015-06-23"
url: "/2014/07/25/calling-signalr-hub-from-a-csharp-wcf-service"
comments: "true"
categories:
  - web development
  - signalr
  - programming
tags:
  - wcf
  - c-sharp
  - iis
  - tutorial
  - chat
series:
---
## Introduction
**UPDATE**: See all my SignalR tutorials [right here](http://systemout.net/categories/signalr/) or find a post about how to list users that are connected [right here](http://systemout.net/2015/06/10/developing-a-who-is-on-chat-web-app-using-signalr-and-wpf/).

Download the source for this tutorial [here](https://drive.google.com/file/d/0B_b7_Dquxu0KWjlCWVJyVUlZbGM/view?usp=sharing).

In a previous blog post I wrote a tutorial on how to develop and deploy a simple SignalR chat application using the bare minimum as a proof of concept. You can find that post right here: [http://systemout.net/2014/07/17/developing-and-deploying-a-signalr-chat-web-app-to-iis/](http://systemout.net/2014/07/17/developing-and-deploying-a-signalr-chat-web-app-to-iis/)

In this tutorial, we are going to make a WCF service that can talk to the SignalR chat app and broadcast messages from "the system" - not by a user.
SignalR is easy to use with JavaScript, but if you need to broadcast messages to the SignalR hub, you need to use a browser (JavaScript) or otherwise things get a little more complicated.
This tutorial, will guide you to calling SignalR hub methods from a WCF service that recides in a different DLL than the SignalR host.<!--more-->

### Setup
This tutorial uses the Chat web app as implemented in the previous mention blog post, you can find the solution from that post rigth here: [http://1drv.ms/1wB5IWF](http://1drv.ms/1wB5IWF)
Oh, and if you want to see the app in action, you can try it out right here: [http://systemout.net:56677/ChatWebApp/Messenger.html?username=User1](http://systemout.net:56677/ChatWebApp/Messenger.html?username=User1)

We are going to add a WCF service to the ChatWebApp solution and provide functionality that allows the WCF service to invoke methods on the SignalR hub.

### Creating the WCF Service
Open up the ChatWebApp solution.
Add a WCF Service Application, name it WcfService:
![](http://systemout.net/images/SignalRCreateWcfServiceProject.png)

Now add the SignalR depencies, right click References > Manage NuGet Packages > Search for SignalR > Install **Microsoft ASP.NET SignalR .NET Client** and **Microsoft ASP.NET SignalR Core Components**.

We also need a reference to the ChatWebApp project, so if you are not building on the ChatWebApp project which you can download in the top of this post, you need to write your own Hub class. But, for this tutorial we are using the ChatWebApp, so add a reference to that project.

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

VS have created the basic files needed to host the WCF Service but I have renamed the IService1.cs and Service1.cs to ISystemService.cs and SystemService.
Replace the code in **SystemService.cs** with:
{{< highlight  csharp >}}

    public class SystemService : ISystemService
    {
        private readonly IHubProxy messageHub;

        public SystemService()
        {
            var connection = new HubConnection("http://localhost:yourport/~/");
            messageHub = connection.CreateHubProxy("MessageHub");
            connection.Start().Wait();
        }

        public void SendMessage(string sender, string message)
        {
            messageHub.Invoke("SendMessage", sender, message);
        }
    }
{{< /highlight >}}
In the constructor of the service were a making a connection to the hub and creating a proxy for the MessageHub in the ChatWebApp project and last we are starting the connection.
Maybe you don't need the ~ at the end of the hub connection URL, but again, if working on the ChatWebApp you do.

The service publishes one operation: SendMessage. I don't think further introduction to the workings are required, **but** it is important the the method we are invoking on the hub is written exactly as it is named in the hub.

Now to the interface; replace the code in the ISystemService.cs whit this:
{{< highlight  csharp >}}

    [ServiceContract]
    public interface ISystemService
    {
        [OperationContract]
        void SendMessage(string sender, string message);
    }
{{< /highlight >}}
And that's it! We are ready to run the service.
Mark the SystemService.cs file and hit F5 to bring up the WCF Test Client GUI the Visual Studio provides where you can invoke the SendMessage operation.

If you want, you can try to connect to the ChatWebApp that I have running, simply create the hub connection like so:
{{< highlight  csharp >}}

	var connection = new HubConnection("http://systemout.net:56677/ChatWebApp/~/");
{{< /highlight >}}
Then you can go to:<a href="http://systemout.net:56677/ChatWebApp/Messenger.html?username=user1" target="_blank"> my demo site</a> and see that the messages you send from your WCF service are send all the way to my server in Denmark and back to your browser :)

![](http://systemout.net/images/ChromeExample.png)

You can download the updated solution including the WCF service here: [http://1drv.ms/1qEpMtQ](http://1drv.ms/1qEpMtQ)

**[You can find my new post about how to create a "Who Is On" SignalR service right here!](http://systemout.net/2015/06/10/developing-a-who-is-on-chat-web-app-using-signalr-and-wpf/)**
