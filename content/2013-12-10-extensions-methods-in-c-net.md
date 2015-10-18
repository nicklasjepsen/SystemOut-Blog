---
title: "'Extension methods in C#/.NET'"
author: "Nicklas Møller Jepsen"
date: "2013-12-10"
url: "/2013/12/10/extensions-methods-in-c-net/"
comments: "true"
categories:
  - programming
tags:
  - c-sharp
  - avoid duplicate code
  - extension method
  - sample code
series:
---
A nice feature in C# is the ability to use extension methods. These methods can be used (as the name suggests) to extend functionality to existing classes.

In my opinion, extension methods can be used to **write cleaner and more easy to read code**. They can also be used to avoid **duplicate code**.  
Take an example; you are writing an application, which needs to determine if an email address is in a valid format (eg. name@domain.com). Now, you could make this check right where the users enters the email or you could create a custom TextBox control which will only accept valid email addresses. That is fine.

But what if another part of your application is using the provided email address but to make the application robust it is surely validating that the email is in fact a valid email address before using it?

<!--more-->

  
The custom TextBox would not be of much use and the code you wrote where the user enters the email address cannot be invoked from the other part of your application.  
Now the extension method comes in handy. Let’s say you can call a method on the string object with the name IsValidEmail – that would be nice. This can be achieved by writing an extension method for a string.  
There are some general rules when writing extension methods:

*   The containing **class **must be declared as **static**
*   The **method** must be **static**
*   The first **parameter** of the method must be declared with **this keyword** and be of the type you want to extent

Oh and the namespace in which you declare the extension method must be used by the class where you are calling the extension from. As an example, try to remove ‘using System.Linq’ from a class and the try to use List.Any() – that won’t compile.

Now for an example here is a simple IsValidEmail extension method implementation.

<pre class="brush: csharp; title: ; notranslate" title="">public static class StringExtensions
    {
        public static bool IsValidEmail(this string input)
        {
            try
            {
                if (string.IsNullOrEmpty(input))
                    return false;
                new System.Net.Mail.MailAddress(input);
                return true;
            }
            catch (FormatException)
            {
                return false;
            }
        }
    }
</pre>

And here is how the extension can be used:

<pre class="brush: csharp; title: ; notranslate" title="">string emailAddress = "someone@domain.com";
            if (emailAddress.IsValidEmail())
                // The string is a valid email
</pre>

Surely there are other ways to achieve the same thing, and sometimes extension methods are not the best way of doing so (C# is object oriented, so you should write OO code which extension methods can tend to abstain a little from), but this post will not go into details about the OO thinking, look for a later post on that subject.

<a href="http://sdrv.ms/1f4XzCH" title="Extension method source code" target="_blank">Download the source code from here.</a>

For further reading about the subject, have a look at <a href="http://msdn.microsoft.com/en-us/library/bb383977.aspx" target="_blank">Microsoft&#8217;s Programming Guide on Extentension Methods</a>.



<div style="font-size:0px;height:0px;line-height:0px;margin:0;padding:0;clear:both">
</div>