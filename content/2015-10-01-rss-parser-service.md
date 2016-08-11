---
title: "Async RSS Parser Service"
author: "Nicklas Møller Jepsen"
date: "2015-10-01"
url: "/2015/10/01/async-rss-parser-service/"
comments: "true"
categories:
  - programming
tags:
  - c-sharp
  - api
  - rss
  - nuget
series:
---
## Introduction
This post is about how you can implement a generic RSS parser that runs as a service, meaning that it keeps parsing the given URLs in a loop. The reason behind this is that I wanted to be able to data mine some of the daily news from the press and therefore I needed to store the articles in my own database.

The result is a Nuget package that can be included in a .NET project<!--more-->:
`Install-Package SystemOut.RssParser`

If you just want to check out the source for the project, then go to [Github where I host the repository](https://github.com/nicklasjepsen/SystemOut.RssService).

If you want to know how I implemented the code, then please read on!

## RSS Parser Implementation
The project consists of several classes; several RSS model base classes that represents the XML model of a RSS feed.

Let's have a look at some typical RSS XML

{{< highlight  xml >}}
<rss version="2.0">
	<channel>
		<title></title>
		<link></link>
		<description></description>
		<language></language>
		<item>
		<title></title>
			<link></link>
			<description></description>
  			<pubDate></pubDate>
  			<comments></comments>
  			<guid isPermaLink="false"></guid>
  			<enclosure url="" type="image/jpeg" length="3930" />
		</item>
	</channel>
</rss>
{{< /highlight >}}

To be able to deserialize this feed we have some classes: BaseRssFeed.cs, BaseRssChannel.cs and BaseRssItem.cs. Those classes are just plain old C# classes with properties - not that intereseting - you can find them [on Github](https://github.com/nicklasjepsen/SystemOut.RssService) if you want to.

The RssDeserializer.cs is more interesting - here is the GetFeed method:

{{< highlight 	csharp >}}
public static T GetFeed<T>(string feedUrl)
{
	if (string.IsNullOrEmpty(feedUrl)) return default(T);
	
	var xs = new XmlSerializer(typeof(T));
	try
	{
		var xmlReaderSettings = new XmlReaderSettings
		{
			DtdProcessing = DtdProcessing.Parse
		};
		T rss;
		using (var reader = XmlReader.Create(feedUrl, xmlReaderSettings))
		{
			rss = (T)xs.Deserialize(reader);
		}
		return rss;
	}
	catch (WebException webException)
	{
		Logger.Log(LogLevel.Error, webException);
		return default(T);
	}
	catch (InvalidOperationException invalidOperationException)
	{
		Logger.Log(LogLevel.Error, invalidOperationException);
		return default(T);
	}
}
{{< /highlight >}}

As you can see, we are using the XmlSerializer and XmlReader to read and deserialize the RSS feed. Not much magic, because we get most by using the built in implementations of .NET.

Now to be able to be able to use the deserializer generically we have a wrapper around the RssDeserializer:

{{< highlight  csharp >}}
public async Task<List<FeedItem>> Parse(FeedSource source)
{
    var feed = await Task.Run(() => RssDeserializer.GetFeed(source.Url));
    var channel = feed?.GetRssChannels()?.FirstOrDefault();
    if (channel == null)
        return new List<FeedItem>();
    return (from rssItem in channel.GetRssItems()
            select new FeedItem
            {
                Title = rssItem.Title,
                Url = rssItem.Link,
                ExternalItemId = rssItem.GetGuid(),
                ImportTime = DateTime.UtcNow,
                PublishTime = rssItem.Date,
                FeedSource = source,
                Summary = rssItem.Description,
            }).ToList();
}
{{< /highlight >}}

This method calls the deserializers parse method and transforms the objects into the desired feed items.

Now to the most intereseting part - the RSS Parsing Service.
The class **NewsFeedParseService.cs** is designed with custom events that will be triggered once a new RSS feed item is imported. This is done so that the caller can determine what to do with the new item. First I had an Entity Framework context in the NewsFeedParserService but when I wanted to publish the solution as a Nuget package I needed to extract the database logic from the parsing. Therefore I decided to create an event based service that notifies the caller when a new item is imported. This moves the responsibility of data persistance away from the parser and out where it belongs.
For the above reason we have the following event i the NewsFeedParseService.cs:

{{< highlight  csharp >}}
public delegate void NewFeedItemHandler(object sender, FeedItemEventArgs e);
public event NewFeedItemHandler OnNewFeedItems;
{{< /highlight >}}
 
 This event is triggered in the Execute method that async runs until cancelled:
 
{{< highlight  csharp >}}
public async Task Execute(CancellationToken cancellationToken)
{
	var feedItemsCache = new HashSet<string>();
	do
	{
		foreach (var feedSource in feedSources)
		{
			Logger.Debug($"Reading feed from {feedSource.Url}.");
			var newItems = new List<FeedItem>();
			var items = await defaultParser.Parse(feedSource);
			if (items == null)
				continue;
		
			foreach (var feedItem in items)
			{
				if (feedItemsCache.Contains(feedItem.ExternalItemId))
					continue;
				feedItemsCache.Add(feedItem.ExternalItemId);
				newItems.Add(feedItem);
			}
		
			Logger.Debug($"Got {newItems.Count} new items.");
		
			if (newItems.Count > 0)
				OnNewFeedItems?.Invoke(this, new FeedItemEventArgs { Items = newItems });
		
		}
		
		Logger.Info($"Sleeping {ConfigurationProvider.GetIntValue(ConfigurationProvider.FeedImportIntervalInSeconds, 60)} second(s) before starting all over.");
		await Task.Delay(ConfigurationProvider.GetIntValue(ConfigurationProvider.FeedImportIntervalInSeconds, 60) * 1000, cancellationToken);
	} while (!cancellationToken.IsCancellationRequested);

	Logger.Info("Task is cancelled - escaping endless loop.");  
}
{{< /highlight >}}

What happens here is that we are creating a HashSet using this a a cache for already imported feeditems. Because RSS parsing is poll based we need to import the manually and prevent the clients from getting multiple events for the same items.
Further more we are calling the parse operation and then iterating all the results and if they do not exist in the cache, we trigger a new event, notifying the caller that a new item is imported.
This goes on in a loop that only gets escaped if the caller decides to cancel the execution.

And that's pretty much it! Next post will be about how to use SignalR to push the new post to clients so we can create a web site that dynamically updates when a new post is added.

Remember to check out the [Nuget site for the project](https://www.nuget.org/packages/SystemOut.RssParser/) or simply install it directly in you projects:
`Install-Package SystemOut.RssParser`

Also check out [the Github repo](https://github.com/nicklasjepsen/SystemOut.RssService)!