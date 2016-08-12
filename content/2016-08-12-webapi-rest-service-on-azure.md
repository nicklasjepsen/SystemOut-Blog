---
title: "Create a WebApi REST Service Hosted on Azure"
author: "Nicklas Møller Jepsen"
date: "2016-08-12"
url: "/2016/08/12/create-a-webapi-rest-service-hosted-on-azure/"
comments: "true"
categories:
  - programming
tags:
  - c-sharp
  - api
  - webapi
  - azure
  - REST 
---
## Introduction


If you just want to check out the source for the project, then go to the [Github repo](https://github.com/nicklasjepsen/WebApiRestServiceExample).

## Craete the WebApi project
Fire up Visual Studio (I use 2015 and the screenshots in this post is based on that).

Create a new project: File > New > Project

![New Project 1](http://systemout.net/images/WebApiRestService/NewProject.png)

Now select Empty and check WebAPI:

![New Project 2](http://systemout.net/images/WebApiRestService/NewProject1.png)

This gives us a good starting point with some default folders scaffolded for our Controllers and Models.

## Implement a Controller
Each controller is a class that can hold our service logic, the functions that we want to provide to clients.
Let's say we wanted to write an API that clients can use get the local time for a given city/country.

Right click the Controllers folder, click Add > Controller... Now select Web API 2 Controller - Empty, click Add and name it CalendarController. (Sometimes it's good to be a little abstract when naming controllers, especially if they are sort of utility controllers that provides different functions).

Now we get a CalendarCOntroller class that extends the **ApiController** class. We will add 2 API methods to this controller; one that takes a city as a parameter and one that both takes city and country as parameters.

Have a look at the code below and I will explain it in details:


{{< highlight 	csharp >}}

	[HttpGet]
	[Route("Calendar/{city}/LocalTime")]
	public async Task<IHttpActionResult> GetTime(string city)
	{
	    return await GetLocalTimeInternal(city, string.Empty);
	}

{{< /highlight >}}

There are some things to note:
1. We're annotating the method with HttpGet to tell WebApi that it should route GET request to here
2. We're setting up the route so that http://server.com/Calendar/Copenhagen/LocalTime will end up using this method
3. The parameter string city is resolved from the route {city}
4. We're returning a IHttpActionResult wrapped in a Task, so our method is async and we can call other methods async. We could just return a string or our own custom object, but by doing this we'll be able to make fully use of WebApi and return the build-in Ok() or NotFound().

## Implementing the Model
As mentioned, we can just return strings, int, etc in WebApi, however that's no fun. Therefore, we'll implement a model to represent the data.
Add a new class to the Model folder in the solution (Ctrl+Shift-C while having the Model folder marked), name it LocalTimeModel.cs and copy in this code:

{{< highlight 	csharp >}}

	public class LocalTimeModel
	{
    	public string TimeZoneName { get; set; }
    	public double UtcOffSet { get; set; }
	    public DateTime LocalTime { get; set; }
	}

{{< /highlight >}}

Now we'll go through the actual implementation of the method.

## Get Local Time From City
We're using Google Maps API for getting the coordinates for a given city and then, again, using Google Maps API to translate those coordinates into the right time zone.
After that is done, it's peace of cake to use the built in libs in .NET to get the current local time.

{{< highlight 	csharp >}}

	private async Task<IHttpActionResult> GetLocalTimeInternal(string city, string country)
	{
	    var timeZoneName = await googleMapsProvider.GetTimeZoneName(city, country);
	    if (string.IsNullOrEmpty(timeZoneName))
	        return NotFound();
	
	    var localTime = dateTimeProvider.GetTimeForTimeZone(timeZoneName);
	
	    return Ok(new LocalTimeModel
	    {
	        TimeZoneName = timeZoneName,
	        UtcOffSet = Math.Round((localTime.ToUniversalTime() - DateTime.UtcNow).TotalHours, 2),
	        LocalTime = localTime
	    });
}
{{< /highlight >}}

I won't enclose the Google Maps API classes here - it's out of the scope of this post, but you'll find them in the [Github repo](https://github.com/nicklasjepsen/WebApiRestServiceExample) together with the rest of the source for this example.

## Wrapping Up!
Now you should be able to hit F5 and append /Calendar/Copenhagen/LocalTime to the http://localhost:{portNumber} in the browser that opens and you will get the current local time in Copenhagen, together with some other time zone infomation.

If not, then please clone my [Github repo](https://github.com/nicklasjepsen/WebApiRestServiceExample) and compare it to your code.

## Thanks for reading!
...and please check out the [Github repo](https://github.com/nicklasjepsen/WebApiRestServiceExample)!

Please post you questions/comments if you have any.