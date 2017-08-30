---
title: "Create a Windows 10 IoT Universal App Photo Viewer with Bing Maps Support"
author: "Nicklas Møller Jepsen"
date: "2015-05-28"
url: "/2015/05/28/create-a-windows-10-iot-universal-app-photo-viewer-with-bing-maps-support/"
comments: "true"
categories:
  - programming
  - raspberry pi
tags:
  - c-sharp
  - iot
  - uwp
  - xaml
series:
---
### Introduction
In this post I will walk your through in creating a photo viewer with Bing Maps integration. The viewer is hosted on a Raspberry Pi 2 running Windows 10 IoT Preview and is created as a C#/XAML Universal Windows Platform (UWP) application.

UWP is a great way of writing code that reached broad - before UWP we could still create a single solution in Visual Studio, but we would need separate DLLs for each platform target - now we can just create one DLL that can be installed on all Windows platforms. That's great!<!--more-->

Here is all the steps required to get going:
- Install Windows 10 Preview
- Install Visual Studio 2015 Preview/Beta/RC/whatever state it currently is in :)
- Install Windows 10 IoT Core Preview on the Pi
- Setup Bing Map Developer account and acquire a security token
- Write the code
- Deploy to the Pi
- ...aaand done!

[Download the solution](https://drive.google.com/file/d/0B_b7_Dquxu0KWjJWd2FTaDlxRms/view?usp=sharing)

![Project screen shot](http://systemout.net/images/PiPicSample.png)

### Installing Windows and VS
There are plenty of good guides on how to install Windows 10 and also on how to install it on the Pi:
A quick note: **You need a physical PC running Windows 10, unfortunately a VM won't do :(**

Here are the links for downloading and installation instructions for all the required software:

- <a href="http://windows.microsoft.com/en-us/windows/preview-faq" target="_blank">Install Windows 10</a>
- <a href="https://ms-iot.github.io/content/en-US/win10/SetupRPI.htm" target="_blank">Install Windows 10 IoT Core</a>
- <a href="https://www.visualstudio.com/en-us/downloads/visual-studio-2015-downloads-vs.aspx" target="_blank">Visual Studio 2015</a>

### Setup Bing Maps
You need to register your Microsoft account as a Bing Developer which can be done using the Bing Portal right here: [Bing Maps Dev Center](https://www.bingmapsportal.com "Bing Maps Dev Center"). Once registered you need to acquire a key/token go to **My Account > Create or view keys**. Follow the link to create a new key, follow the guide and remember to select Basic / Universal Windows App as the key type.

Copy the key, you will need to enter it later when creating the application.

### Writing the code
The solution is nice and simple - all we need is a single view and some C# classes.
The consists of a couple of text blocks, an image view and a Map control:
{{< highlight  xml >}}
	<Grid Background="Black" Grid.Column="1">
        <Image x:Name="imageControl" Margin="0,0,237,0"/>
        <Maps:MapControl x:Name="MapControl" MapServiceToken="TOKEN" Height="202" VerticalAlignment="Bottom" Margin="0,0,24,25" HorizontalAlignment="Right" Width="213"/>
        <TextBlock x:Name="DetailsTxb" Margin="0,10,-9,0" TextWrapping="Wrap" Text="TextBlock" Foreground="White" Height="20" VerticalAlignment="Top" FontWeight="Bold" FontSize="16" HorizontalAlignment="Right" Width="241"/>
        <TextBlock x:Name="MoreDetailsTxb" Margin="0,35,10,0" TextWrapping="Wrap" Text="TextBlock" Foreground="#FFB8B8B8" Height="20" VerticalAlignment="Top" HorizontalAlignment="Right" Width="222"/>
    </Grid>
{{< /highlight >}}

Then we need some magic to read the photos from disk, read properties from each photo file like latitude and longitude, translate the coordinates using the Bing Maps service, update the text blocks with the data and finally show the image and set the location of the Map control. All this is implemented in C# in the code behind file.

Here's how to get the image files:

{{< highlight  csharp >}}
KnownFolders.PicturesLibrary.GetFolderAsync("PiPic");
{{< /highlight >}}

This will look in the current users Pictures library for a folder named PiPic. Note that in the UWP there are no File library or other of that stuff we are used to from .NET - this is because that UWP will work on a phone also where we not necessarily have a C: drive.

##### Reading the image properties

{{< highlight  csharp >}}
var propsTask = file.Properties.GetImagePropertiesAsync();
var props = await propsTask;
var image = new ImageDto();
image.Location = new LocationDto
	{
    	Latitude = (double)props.Latitude,
        Longitude = (double)props.Longitude,
    };
{{< /highlight >}}

The ImageDto and LocationDto are classes I have created for easability, download the complete solution to get the big picture.

Now with the coordinates in hand we can translate those to a location:
{{< highlight  csharp >}}
private async Task<string> ReverseGeocode(LocationDto location)
{
    // Location to reverse geocode.
    BasicGeoposition basicLocation = new BasicGeoposition();
    basicLocation.Latitude = location.Latitude;
    basicLocation.Longitude = location.Longitude;
    Geopoint pointToReverseGeocode = new Geopoint(basicLocation);

    // Reverse geocode the specified geographic location.
    MapLocationFinderResult result =
        await MapLocationFinder.FindLocationsAtAsync(pointToReverseGeocode);

    // If the query returns results, display the name of the town
    // contained in the address of the first result.
    if (result.Status == MapLocationFinderStatus.Success)
    {
        return result.Locations[0].Address.Town +
            ", " + result.Locations[0].Address.Country;
    }

    return string.Empty;
}
{{< /highlight >}}

I found this code somewhere on MSDN (can't remember the link) so I can't take credit for that, but it works :)
Basically we are just querying the Bing Maps service and returning the result.

##### Updating the Map control
Here's how to center the Map control to the location of choice:
{{< highlight  csharp >}}
MapControl.Center = new Geopoint(new BasicGeoposition()
{
	Latitude = image.Location.Latitude,
    Longitude = image.Location.Longitude
});
MapControl.ZoomLevel = 17;
MapControl.LandmarksVisible = true;
MapControl.MapElements.Clear();
{{< /highlight >}}

#### Deploying to Windows 10 on Raspberry Pi 2
Now this is easy! All we have to do is change the debug option in Visual Studio to ARM and select Remote Machine and enter the IP adddress of the Pi. Remember to change authentication mode to None (default is Windows). Also, change the build type to Release - then you get rid of the diagnostics numbers in the top corners, but this heavily increases the build time.

For more info on deploying an UWP to Windows 10 have a look [here!](http://ms-iot.github.io/content/en-US/win10/samples/HelloWorld.htm) Look for the paragraph "Deploy the app to your Windows IoT Core device" This guide also contains instructions on how to set the application as the startup application. By doing so you can plug the photo Pi to your TV and once powered on you will have your very own photo viewer on the big screen!

And that's it - remember you can find the **complete solution right [here!](https://drive.google.com/file/d/0B_b7_Dquxu0KWjJWd2FTaDlxRms/view?usp=sharing)**
