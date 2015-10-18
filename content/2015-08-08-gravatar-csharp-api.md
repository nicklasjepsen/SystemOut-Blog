---
title: "Gravatar C# API"
author: "Nicklas Møller Jepsen"
date: "2015-08-08"
url: "/2015/08/08/gravatar-csharp-api/"
comments: "true"
categories:
  - programming
tags:
  - c-sharp
  - api
  - gravatar
  - nuget
series:
---
Each time I write a new app that involves users (which is pretty often) I think it would be nice to be able to display their own profile image, without the need of authenticating them against facebook, Linkedin or by using their Microsoft account.

Therefore I decided to use Gravatar, which basically allows you to request a user image (avatar) when you only know their email address.<!--more--> This is pretty simply and you can just make a plain HTTP GET and get their image url in response. Try this:
 
`http://gravatar.com/avatar/f561d9206d313b49f9bde3bd50803b84`

If you don't want to read any more and just want the Nuget package: 

`Install-Package GravatarSharp.Core`

Now, there is now email address in this request, and that is because Gravatar uses a hash of the users email. So you would need some way of generating the hash from the users email and then making the request. 
Further more you can retrieve lots of other information about the users presented in JSON format:

`http://en.gravatar.com/nicklasjepsen.json]`
    
Because I write many applications that could use this, and because I love small APIs/libraries I decided to make a Gravatar C# API and put the code on Github and make the API distributable using Nuget.

You can find the project here [on Github](https://github.com/nicklasjepsen/GravatarSharp)

And on Nuget: [GravatarSharp.Core](https://www.nuget.org/packages/GravatarSharp.Core)

If you just need to install the Nuget package for a project, simply open the package manager and enter:

`Install-Package GravatarSharp.Core`