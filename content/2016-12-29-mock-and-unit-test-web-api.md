---
title: "Mock and Unit Test a WebApi REST Service"
author: "Nicklas Møller Jepsen"
date: "2016-12-29"
url: "/2016/12/29/mock-and-unit-test-web-api/"
comments: "true"
categories:
  - programming
tags:
  - c-sharp
  - api
  - webapi
  - unit-test
  - mock
  - moq
  - unity
  - di 
---
## Introduction
This post extends my previous post on how to create a ASP.NET WebApi, please see: [Create a WebApi REST Service Hosted on Azure](http://systemout.net/2016/08/12/create-a-webapi-rest-service-hosted-on-azure/).

If you want to jump right in, then feel free to get the source for the WebApi posts [here on Github](https://github.com/nicklasjepsen/WebApiRestServiceExample).

**High-level solution description:**

* REST service that takes a city/country and returns the date time and timezone
* Using GoogleMaps API to lookup the information

###  Determine what to test
Now in this rather simple example, in the WebApi controller we really only have some real logic in the `GetLocalTimeInternal` method, so this will be an obvious choice to target our first unit tests.

**A word on unit testing**: I often find the most challenging part to be to determine and seperate the code that I need to test. For instance, in this example we use Google Maps Api to get the time zone from a city/country and we are really not interested in testing the Google Maps Api (I'm sure the folks at Google is quite capable of doing that!). What we want to test is that our service is calling the Google Maps Api with the parameters we expect, and that our service/controller is able to understand the response and also handle any errors returned.

### How and why to mock
Once we figure out what to test, we need to isolate this code for easy testing. We can do this by follow simple object oriented principles as the separation of concerns pattern and let our controller handle the web part and split the other logical parts of the solution into separate classes. This will give us the option of injecting these parts into the controller thus giving us the option of mocking those parts.

Let's take a look! Here's what we have from the previous post:

{{< highlight 	csharp >}}

	public class CalendarController : ApiController
    	{
       	private readonly GoogleMapsProvider googleMapsProvider;
        	private readonly DateTimeProvider dateTimeProvider;
    
        	public CalendarController()
        	{
            		googleMapsProvider = new GoogleMapsProvider();
            		dateTimeProvider = new DateTimeProvider();
        	}
        
        ...
 {{< /highlight >}}

We can see that we have a GoogleMaps provider and a DateTime provider that we use in the `CalendarController`. To be able to mock these and by that be able to control what they return so that we can test the relevant code, we need to wrap those classes in interfaces. This will let us use Moq, which is a mocking framework that easy our life when it comes to unit testing.

So we create the following interfaces:

{{< highlight 	csharp >}}
 
	public interface IGoogleMapsProvider
    	{
       	Task<string> GetTimeZoneName(string city);
        	Task<string> GetTimeZoneName(string city, string country);
    	}
 
 {{< /highlight >}}
 
 {{< highlight 	csharp >}}
 
	public interface IDateTimeProvider
    	{
       	DateTime GetTimeForTimeZone(string timeZoneName);
    	}
 
 {{< /highlight >}}

We should also inject these into the `CalendarController` constructor and change the type of the local variables to be the interface type:

 {{< highlight 	csharp >}}
 
    	public class CalendarController : ApiController
    	{
       	private readonly IGoogleMapsProvider googleMapsProvider;
        	private readonly IDateTimeProvider dateTimeProvider;

       	public CalendarController(IGoogleMapsProvider googleMapsProvider, IDateTimeProvider dateTimeProvider)
        	{
            		this.googleMapsProvider = googleMapsProvider;
            		this.dateTimeProvider = dateTimeProvider;
        	}
 
 {{< /highlight >}}
 
By doing this, we will be able to inject the mocks from our unit tests.

**We have one issue thou: WebApi expects a parameterless constructor**

### Setup Unity
To resolve this issue, we can use the Unity.WebAPI nuget from Microsoft. This is a DI framework that can inject the classes into our controller and make WebApi happy again.
Open the package manager console and run `install-package unity.WebAPI` follow the steps in the readme.txt that opens after the nuget is installed to ensure that unity is properly configured.

#### Registering our classes
We need to tell Unity which type to inject into the `CalendarController`. We do this in the `UnityConfig` class. Here's how it should look:

 {{< highlight 	csharp >}}
 
	public static class UnityConfig
    	{
       	public static void RegisterComponents()
        	{
			var container = new UnityContainer();
            
            		// register all your components with the container here
            		// it is NOT necessary to register your controllers
            
            		// e.g. container.RegisterType<ITestService, TestService>();
            		container.RegisterType<IGoogleMapsProvider, GoogleMapsProvider>();
            		container.RegisterType<IDateTimeProvider, DateTimeProvider>();

            		GlobalConfiguration.Configuration.DependencyResolver = new UnityDependencyResolver(container);
        	}
    	}
 
 {{< /highlight >}}

Now run your solution and try calling the CalendarController: [http://localhost:PORTNUMER/Calendar/Copenhagen/LocalTime](http://localhost:PORTNUMBER/Calendar/Copenhagen/LocalTime)

### Testing and Mocking
Finally we are able to begin writing our tests!

* Add a new UnitTest project to the solution: ![New Project 2](http://systemout.net/images/WebApiRestService/NewUnitTestProject.png)
* Install Moq nuget in the unit test project: install-package Moq (remember to select the unit test project in the package manager console)
* Create a unit test class and test the functionality

 {{< highlight 	csharp >}}
 
   	[TestClass]
   	public class CalendarControllerUnitTests
    	{
       	[TestMethod]
        	public async Task TestCorrectParametersToGoogleMaps()
        	{
            		const string city = "Copenhagen";
            		var gmMock = new Mock<IGoogleMapsProvider>();
            		var dtMock = new Mock<IDateTimeProvider>();

            		// Need to call setup once
			gmMock.Setup(m => m.GetTimeZoneName(It.IsAny<string>(), It.IsAny<string>())).Returns(Task.Run(() => ""));

            		var controller = new CalendarController(gmMock.Object, dtMock.Object);

            		await controller.GetTime(city);

            		// Verify that GoogleMaps are called with the value that the client is passing along (we are only passing city, so country is null)
            		gmMock.Verify(m => m.GetTimeZoneName(city, null));
        	}
    	}
 
 {{< /highlight >}}
 
This method only tests that the GoogleMaps api is called with the correct city. We are not validating the result or anything here and we are not testing the GoogleMaps api implementation.

Now we want to test that we handle the result correctly and that is translates to a response that the client understands.
Add the following unit tests:

 {{< highlight 	csharp >}}
 
 	[TestMethod]
      	public async Task TestValidResponseReturned()
      	{
       	const string city = "Copenhagen";
            	var gmMock = new Mock<IGoogleMapsProvider>();
            	var dtMock = new Mock<IDateTimeProvider>();

            	// Need to call setup once
            	gmMock.Setup(m => m.GetTimeZoneName(It.IsAny<string>(), It.IsAny<string>())).Returns(Task.Run(() => "Central European Standard Time"));

            	var controller = new CalendarController(gmMock.Object, dtMock.Object);

            	var response = await controller.GetTime(city) as OkNegotiatedContentResult<LocalTimeModel>;
            	// Assert that we get an OK response and that the LocalTimeModel is not null
            	Assert.IsNotNull(response);
            	Assert.IsNotNull(response.Content);
      	}
 
 {{< /highlight >}}

Now to above tests is asserting that Copenhagen translates to Central European Standard Time and then that the `CalendarController` returns a valid `LocalTimeModel`.

### Add more tests
Now we have a solution set up for unit testing and now it's just a matter of writing more tests, which I'll leave up to you! Remember it's not a question of how much of your code that gets tested. The important things is to test the *right* code!

### Final words
Remember you can get the source code [here on Github](https://github.com/nicklasjepsen/WebApiRestServiceExample).
And here's the first part that this post is based upon: [Create a WebApi REST Service Hosted on Azure](http://systemout.net/2016/08/12/create-a-webapi-rest-service-hosted-on-azure/).
If you are using EntityFramework then you might find my [post on testing EF](http://systemout.net/2014/01/02/testable-data-access-layer-using-entity-framework-6/) relevant.

Thanks for reading and please let me know your thoughts and suggestions in the comments just below this post!