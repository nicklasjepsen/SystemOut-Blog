---
title: "How To Measure Execution Time and Performance Optimize Source Code"
author: "Nicklas Møller Jepsen"
date: "2014-08-19"
url: "/2014/08/19/measure-execution-time-and-performance-optimize-source-code"
comments: "true"
categories:
  - programming
tags:
  - tools
  - test
  - time
  - iterate
  - c-sharp
series:
---
## Why Should You Measure Your Source Code?
I work by the princip; don't optimize until needed. Which means that I write the code, make sure it does what it is supposed to, compile, test and release and **then if** a bottleneck is found, I try to optimize it.

But before I'm able to optimize, I need to be able to measure for the simple reason that otherwise I'm not able to determine wheter or not the optimization actually hard an inpact and most importantly; that it did not have a negative impact on the performance.

For that simple reason, I have written a small component that is able to measure the time a specific operation takes to complete. The component uses the [Stopwatch class](http://msdn.microsoft.com/en-us/library/system.diagnostics.stopwatch.aspx "Stopwatch class on MSDN"), which seemed to be a good choise for the purpose, but I must say I haven't made througly analasys of wheter this class is the best suited for the job.

### Results
It's always a good idea to prove the importance of the contents of a post, before actually writting the post, so here are some simple results (and also reasons) for code optimization:

<table>
    <tr>
        <td><b>Method</td>
        <td><b>Iterations</td>
		<td><b>Average in miliseconds</td>
    </tr>
    <tr>
        <td>MaxFactor1 for 123456789</td>
		<td>50</td>
        <td>557,48</td>
    </tr>
    <tr>
        <td>MaxFactor2 for 123456789</td>
		<td>50</td>
        <td>0,04</td>
    </tr>
</table>  
<!--more-->
Basicly I have taken the samples from [this StackOverflow post](http://stackoverflow.com/questions/2535251/c-finding-the-largest-prime-factor-of-a-number) and measured 2 of the solutions, see for clarity I have included the Maxfactor1 and 2 method at the bottom of this post.

The point of the above table is to show the big difference in the performance of the 2 Maxfactor methods - when run 50 times the average time taken is much faster in the Maxfactor2 method.


## The Measurement Component
You can download the source/solution [here](http://1drv.ms/1qn4PNM "Solution on OneDrive").

I wanted to make an easy to use component with minimal impact on the actual code that do the main stuff in the application.
To accomplish this, I have written a **Time** method:
	        
	public void Time(Action actionToTime, int numberOfRuns, string actionIdentifier)
        {
            var sw = new Stopwatch();
            var resultSet = GetResultSet(actionIdentifier);
            for (var i = 0; i < numberOfRuns; i++)
            {
                var executionTime = DateTime.Now;
                sw.Start();
                actionToTime();
                sw.Stop();

                resultSet.Results.Add(new Result
                {
                    Elapsed = sw.ElapsedMilliseconds,
                    TimeOfExecution = executionTime,
                });

                sw.Reset();
            }
        }

Which can be invoked like this:

	    [TestMethod]
        public void TestDownloadString()
        {
            var tm = new TimeMeasurement();
            tm.Time(() => DownloadString("http://systemout.net"), 50, "DownloadString");

            tm.FlushResultsAsync().Wait();
        }
	
		public void DownloadString(string url)
        {
            using (var web = new WebClient())
            {
                web.DownloadString(new Uri(url));
            }
        }

Now the thing that's happening in the Time method is that we're using a Stopwatch to measure the execution time of the method. Furthermore we're adding a ResultSet (see below) for the given actionIdentifier. This is used to keep track of the results for the given actionIdentifier so it later can be flushed to a file:

	    public async Task FlushResultsAsync()
        {
            // First get a copy of the results so no modifications are made
            // while we process the results.
            var results = resultSets.Values.ToList();
            resultSets.Clear();

            await writer.WriteResultSetsAsync(results);
        }

The writer is for the time being a CSV text writer implementation. I plan on making it an IResultWriter so that the caller can provide it's own writer implementation to control how the timing output is stored - for now it is written to a file in C:\Users\USERNAME\AppData\Roaming\SystemOut\Performance

You can download the complete source including the solution/project files [here](http://1drv.ms/1qn4PNM "Solution on OneDrive").

## Maxfactor Methods
I'm not a math genious, but from the below source it seems that it's much faster to go from largest number and down then the other way around. Obviously. :)

        static private long Maxfactor1(long n)
        {
            for (var k = n / 2; k > 1; k--)
            {
                if (n % k == 0 && Prime(k))
                {
                    return k;
                }
            }

            return 1;
        }

        static private long Maxfactor2(long n)
        {
            long k = 2;
            while (k * k <= n)
            {
                if (n % k == 0)
                {
                    n /= k;
                }
                else
                {
                    ++k;
                }
            }

            return n;
        }

        static private bool Prime(long x)
        {
            for (long i = 2; i <= x / 2; i++)
            {
                if (x % i == 0)
                    return false;
            }
            return true;
        }