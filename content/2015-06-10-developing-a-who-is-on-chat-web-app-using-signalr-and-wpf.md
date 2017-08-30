---
title: "Developing a 'Who Is On' Chat Web App Using SignalR and WPF"
author: "Nicklas Møller Jepsen"
date: "2015-06-10"
url: "/2015/06/10/developing-a-who-is-on-chat-web-app-using-signalr-and-wpf/"
comments: "true"
weight: 20
categories:
  - programming
  - signalr
tags:
  - wpf
  - c-sharp
  - chat
series:
---

## Introduction
In this tutorial I will guide you in creating a "Who Is On" client in Windows Presentation Foundation, WPF. The server is an ASP.NET SignalR app. This post is building upon my previous post about how to develop, deploy and host a SignalR Chat Web App. You can find that post [right here!]({% post_url 2014-07-17-developing-and-deploying-a-signalr-chat-web-app-to-iis %} )<!--more-->

Because we are building upon the previous post and solution, you should download the solution [here](https://drive.google.com/file/d/0B_b7_Dquxu0KWjlCWVJyVUlZbGM/view?usp=sharing).

You can find the source for the WPF app that we are creating in this post [rigth here!](https://drive.google.com/file/d/0B_b7_Dquxu0KR1J0LXRyLXhjalU/view?usp=sharing)

We are going to add some methods to the server side and make an all new (horrible looking) WPF client - this is a SignalR post, not a "make some shiny WPF magic" post.

### The ASP.NET SignalR Backend
In the MessageHub we are going to need to handle users logging on/off. There fore add to the MessageHub.cs the following ConcurrentDictionaries:

{{< highlight  csharp >}}
public static ConcurrentDictionary<string, string> Usernames = new ConcurrentDictionary<string, string>();
public static ConcurrentDictionary<string, string> UserList = new ConcurrentDictionary<string, string>();  
{{< /highlight >}}

Now modify the Login method in the MessageHub class so that it checks if the username provided is in use, and if not, adds the username to the connected clients list:

{{< highlight  csharp >}}
public bool Login(string username)
{
    if (Usernames.ContainsKey(username))
        return false;
    Usernames.TryAdd(username, Context.ConnectionId);
    UserList.TryAdd(Context.ConnectionId, username);

    // Notify all clients that a user is logged in
    Clients.All.showConnected(UserList);

    return true;
}        
{{< /highlight >}}

We need also to be able to remove a user that is logged off/disconnected. This is done in the RemoveUser method:

{{< highlight  csharp >}}
private void RemoveUser()
{
    string username;
    string connectionId;
    UserList.TryRemove(Context.ConnectionId, out username);
    if (!string.IsNullOrEmpty(username))
    	Usernames.TryRemove(username, out connectionId);
}
{{< /highlight >}}

The RemoveUser method is called from multiple methods of the MessageHub, in a public Logoff method that clients can call:

{{< highlight  csharp >}}
public void Logoff(string username)
{
    RemoveUser();
    Clients.All.showConnected(UserList);
}
{{< /highlight >}}

...and also in some overridden methods that handles SignalR events when users are disconnected:

{{< highlight  csharp >}}
public override Task OnDisconnected()
{
    RemoveUser();
    return Clients.All.showConnected(UserList);
}

/// <summary>
/// Needed as of SignalR 2.1.0: http://stackoverflow.com/questions/24878187/signalr-detecting-alive-connection-in-c-sharp-clients
/// </summary>
/// <param name="stopCalled"></param>
/// <returns></returns>
public override Task OnDisconnected(bool stopCalled)
{
    RemoveUser();
    return Clients.All.showConnected(UserList);
}
{{< /highlight >}}

Finally, we add a GetActiveClients method that is used by the clients to trigger SignalR in sending a list of connected clients. Note that currently the methods takes a "requester" as parameter - this is for future use, you can ignore it for now. The method is not quite "sane" ATM because it actually triggers the SignalR event for all connected clients, because we are yet to see how we can send events to specific SignalR clients - look for a near future post about that one :)

{{< highlight  csharp >}}
public void GetActiveClients(string requster)
{
    Clients.All.showConnected(UserList);
}
{{< /highlight >}}

Now that was the server side part. Now we continue to the client.

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

### Implementing the SignalR client int WPF
Now create a new solution/project and make an WPF app.
We are going to use the Signal .NET client in our WPF project. Open the package manager (I usually uses Ctrl+Q, enter package, select first item in list and Enter). Enter Install-Package Microsoft.AspNet.SignalR.Client and hit enter. 
With that done, we can connect our WPF app to the ASP.NET SignalR service hosting our MessageHub. I have assembled all the server communication in a class called ChatService.cs. The class has a couple of public methods like Login, Logoff, StartSignalR, etc. It also holds some events that our UI can subscribe to, so that it can update when for instances receiving a new message or when a user/client is connected/disconnected.

Here is the ChatService.cs (look below the code for a description of how the code works):

{{< highlight  csharp >}}

    using System;
    using System.Collections.Concurrent;
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading.Tasks;
    using Microsoft.AspNet.SignalR.Client;
    using WhoIsOn.Client.ViewModels;
    
    namespace WhoIsOn.Client
    {
        public class ChatService
        {
            private IHubProxy chatHubProxy;
            private string currentUsername;
    
            public event EventHandler<MessageViewModel> OnMessageReceived;
            public event EventHandler<List<UserViewModel>> OnConnectedUsersChanged;
    
            public async Task StartSignalR()
            {
                var connection = new HubConnection("http://localhost:32986/~/");
                //var connection = new HubConnection("http://systemout.net:56677/ChatWebApp/~/");
    
                chatHubProxy = connection.CreateHubProxy("MessageHub");
                chatHubProxy.On<string, string>("displayMessage", OnUserMessage);
                chatHubProxy.On<ConcurrentDictionary<string, string>>("showConnected", ShowConnected);
                await connection.Start();
            }
            private void ShowConnected(ConcurrentDictionary<string, string> userList)
            {
                FireOnConnectedUsersChanged(userList.Values.ToList());
            }
    
            private void OnUserMessage(string username, string message)
            {
                FireOnMessageReceived(username, message);
            }
    
            private void FireOnMessageReceived(string username, string message)
            {
                if (OnMessageReceived != null)
                    OnMessageReceived(this, new MessageViewModel { Message = message, Username = username });
            }
    
            private void FireOnConnectedUsersChanged(IEnumerable<string> users)
            {
                if (OnMessageReceived == null)
                    return;
                var usersViewModel = users.Select(user1 => new UserViewModel
                {
                    Username = user1, Status = "Online"
                }).ToList();
    
                OnConnectedUsersChanged(this, usersViewModel);
            }
    
            public async Task<bool> LoginAsync(string username)
            {
                currentUsername = username;
                return await chatHubProxy.Invoke<bool>("Login", new object[] { currentUsername });
            }
    
            public async Task RequestConnectedClientList()
            {
                await chatHubProxy.Invoke("GetActiveClients", currentUsername);
            }
    
            public async Task<bool> SendMessage(string message)
            {
                return await chatHubProxy.Invoke<bool>("SendMessage", new object[] { currentUsername, message });
            }
    
            public async Task Logoff()
            {
                await chatHubProxy.Invoke("Logoff", currentUsername);
            }
        }
    }
{{< /highlight >}}

The events in the ChatService is used so that the WPF Windows can be updated - there should be no trouble here!
Then there is the StartSignalR method. First we are creating a new HubConnection giving the url where the ASP.NET SignalR app is hosted - if you run the sample from the previous post (donwload link in the top of this post) you should be able to connect to SignalR using the localhost address. Other wise, you can use my SignalR service by uncommenting the second HubConnection line in the StartSignalR method.

Next we create the HubProxy. The string/name must match the name of the class in the ASP.NET SignalR service, in our case that is MessageHub.

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

Then we create eventlisteners using the chatHubProxy.On methods. Here we must provide the name for the methods that, again, must correspond to dynamically created messages defined in the MessageHub methods in the ASP.NET SignalR app. For instance: Clients.All.showConnected(UserList) - here we use showConnected as an identifier. The signature of the delegate methods that we provide must also match. 

The rest of the class is "just" implementation of the methods that will be invoked by SignalR and also triggering of the events to update the UI.

You can download the source for the WPF client [here](http://1drv.ms/1IFqRKj).

Please leave any comments/questions you might have!


