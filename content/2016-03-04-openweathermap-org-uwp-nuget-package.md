---
title: "OpenWeatherMap.org UWP Nuget package"
author: "Nicklas Møller Jepsen"
date: "2016-03-04"
url: "/2016/03/04/openweathermap-org-uwp-nuget-package/"
comments: "true"
categories:
  - programming
tags:
  - c-sharp
  - api
  - openweathermap.org
  - nuget
---
## Introduction
For a hobby project where I created a "Magic Mirror" - more on that later on - I needed some weather information. I decided to go with [OpenWeatherMap](http://openweathermap.org/) because they have a free plan, they have a nice HTTP API and a pretty simple model.

I thought that it probably wouldn't be the last time I would need to include weather data in one of my projects and there fore decided to write a generic Nuget package as a C# Universal Windows Platform class library. The reason for choosing UWP was that I needed to run the application on Windows Core IoT and also to make the Nuget package as usable as possible in future projects.
<!--more-->
You can install the package in your own projects by running the following from the Package Manager Console in Visual Studio:

`Install-Package SystemOutWeatherApi`

If you just want to check out the source for the project, then go to [Github where I host the repository](https://github.com/nicklasjepsen/SystemOutWeatherApi).

## How to use the Nuget
Simply create a new instance of the `WeatherService` and provide the appId that you can acquire from [http://openweathermap.org/api](http://openweathermap.org/api)

Now some notes about the implementation...

## Paste as JSON in Visual Studio
First of I want to share the brilliant little function the devs at the Visual Studio team has provided us with: Paste as JSON, found in the Edit > Paste Special menu in VS.
The reason I'm telling you this, is because in this project, I needed to create a C# representation of the OWM data returned from their service, and that was pretty straight forward with this function.

### Note about localization
The Nuget currently only supports en-US and da-DK locales, but if you want to, go ahead an translate the strings in the Asserts folder in the solution. And if you do, please make a pull request so others can enjoy.

## The interface
The Nuget is just a simple interface wrapping some HTTP requests and de-serializing the JSON into .NET objects, here is a view of the code:

{{< highlight 	csharp >}}
public async Task<WeatherData> GetWeatherDataForCoordinates(string lon, string lat)
{
    return await weatherServiceProvider.ExecuteAsync($"{Uri}lat={lat}&lon={lon}");
}
public async Task<WeatherData> GetWeatherDataForCity(string cityName)
{
    return await weatherServiceProvider.ExecuteAsync($"{Uri}q={cityName}");
}

public async Task<WeatherData> GetWeatherDataForCity(string zip, string country)
{
    return await weatherServiceProvider.ExecuteAsync($"{Uri}zip={zip},{country}");
}

public async Task<WeatherData> GetWeatherDataForCityId(string cityId)
{
    return await weatherServiceProvider.ExecuteAsync($"{Uri}id={cityId}");
}
{{< /highlight >}}

Further then that it is just a matter of exception handling and making sure that the OWP API key is provided.

Remember to check out the [Nuget Site for the project](https://www.nuget.org/packages/SystemOutWeatherApi/) or simply install it directly in you projects:
`Install-Package SystemOutWeatherApi`

Also check out [Github repo](https://github.com/nicklasjepsen/SystemOutWeatherApi)!

Thanks for reading and please post you questions/comments if you have any.