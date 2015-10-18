---
title: "Generic Retry Function to Avoid Duplicate Code"
author: "Nicklas Møller Jepsen"
date: "2013-12-07"
url: "/2013/12/07/retryfunction/"
comments: "true"
categories:
  - programming
tags:
  - c-sharp
  - avoid duplicate code
  - Design patterns
  - sample code
series:
---
Sometimes you have a method which you want to be executed again if it fails the first time.  
This could lead to duplicate code, or to at least to a for loop wrapping the implementation of the method which is OK but what if you had two methods you needed this retry logic for?

<!--more-->

You simply make a method which takes a the method you need to be executed with retry as a parameter. Like this:

<pre class="brush: csharp; title: ; notranslate" title="">private static string ExecuteWithRetry(Func&lt;MethodResult&gt; method, int maxRetries)
        {
            Console.WriteLine("Executing operation.");
            for (var iteration = 1; iteration &lt;= maxRetries; iteration++)
            {
                var result = method();
                _numberOfRuns++;
                Console.WriteLine("Operation executed {0} times.", _numberOfRuns);
                if (result.Success)
                    return result.Message;

                // No luck yet - continue the loop
            }

            throw new Exception(string.Format("Method ran {0} times but did not return successful response.", RetryMax));
        }
</pre>

Now how would you invoke this method?

<pre class="brush: csharp; title: ; notranslate" title="">var message = ExecuteWithRetry(MethodToRun, numberOfRetries);
</pre>

That&#8217;s easy :)

If you want to see the full implementation you can get it <a href="http://sdrv.ms/18rLfuW" target="_blank">here!</a>

Please feel free to write a comment if you have any questions.



<div style="font-size:0px;height:0px;line-height:0px;margin:0;padding:0;clear:both">
</div>