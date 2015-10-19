---
title: "Windows 10 - How to configure network adapter - properties not working"
author: "Nicklas Møller Jepsen"
date: "2015-08-12"
url: "/2015/08/12/windows-10-how-to-configure-network-adapter-properties-not-working/"
comments: "true"
categories:
  - tips and tricks
tags:
  - windows10
  - windows
  - network
  - hotfix
series:
---
So I have installed Windows 10 RTM on my main PC and overall I'm happy with it. Although it still seems a little bit "beta" I am satisfied.
One problem I have encountered is that when setting up a VPN to be able to access company resources I am no longer able to access the Advanced Properties of the VPN Network Adapter<!--more-->:

![VPN Properties](http://systemout.net/images/VPNProperties.png)


Whenever I click on the Properties button for Internet Protocol Version 4 (TCP/IPv4) nothing happens!
This is really annoying me, because I need to be connected to the VPN so that I can access all the internal resources on my workplace, but I don want to use their gateway to route all my traffic hence I need to be able to configure the TCP/IP properties.

The issue is reported [here on Microsoft Answers](http://answers.microsoft.com/en-us/insider/forum/insider_wintp-insider_web/network-connection-vpn-ipv4-properties-not-working/a60edf99-4b06-4219-bb75-b7c08de4ff9e).

### The Solution
As mentioned in the above link, the network connection properties are stored in a plain text file located at "C:\Users\[YOURUSER]\AppData\Roaming\Microsoft\Network\Connections\Pbk\rasphone.pbk"
And the setting for "Use default gateway on remote network" is called "IpPrioritizeRemote".
So changing this to: `IpPrioritizeRemote=0` did indeed fix the problem for me.