---
title: "Windows Phone RSS Reader"
author: "Nicklas Møller Jepsen"
date: "2011-12-10"
url: "/2011/12/10/windows-phone-rss-reader/"
comments: "true"
categories:
  - programming
tags:
  - xaml
  - c-sharp
  - wp7
  - Reader
  - RSS
  - SilverLight
  - SL
series:
---
As part of a competition to win a Windows Phone I have created 3 WP7 apps. One of these is a RSS reader made specific for the danish it news site <a title="Version2" href="http://www.version2.dk" target="_blank">Version2</a>

<!--more-->The source for this application can be found on 

<a href="http://version2viewer.codeplex.com/" target="_blank">Codeplex</a>

The app is based on the <a href="http://msdn.microsoft.com/en-us/library/ff941098(v=vs.92).aspx" target="_blank">Pivot control</a> in WP7. In short this means that you have a page with a list of feed items. On the same page there is a &#8216;tab&#8217; to another page where another feed&#8217;s items are visible:

<p style="text-align:center;">
  <a href="http://systemout.net/wp-content/uploads/2011/12/v2-screen1.png"><img class=" wp-image-53 aligncenter" title="V2-screen1" src="http://systemout.net/wp-content/uploads/2011/12/v2-screen1.png" alt="" width="241" height="438" /></a>
</p>

##  The View

The view is XAML code and is implemented using the <a href="http://en.wikipedia.org/wiki/Model_View_ViewModel" target="_blank">MVVM</a> pattern. This pattern is extremely powerful when it comes to binding data to the view. Furthermore DataTemplates are used to reuse the layout of the feed items.

Here is an example of a DataTemplate:

<pre class="brush: xml; title: ; notranslate" title="">&lt;DataTemplate x:Name="FeedItemTemplate"&gt;
         &lt;StackPanel Orientation="Vertical"&gt;
             &lt;TextBlock Text="{Binding TruncatedTitle}" 
                        TextWrapping="Wrap" 
                        Margin="12,0,0,0" 
                        Style="{StaticResource PhoneTextTitle3Style}"/&gt;
             &lt;StackPanel Orientation="Horizontal" 
                         Margin="0,0,0,17"&gt;
                 &lt;TextBlock Text="{Binding TimeStamp}" 
                            Foreground="#B0D011" 
                            Margin="12,-6,0,0" 
                            Style="{StaticResource PhoneTextSubtleStyle}" /&gt;
                 &lt;TextBlock Text="{Binding Url}"  
                            TextWrapping="NoWrap" 
                            Margin="12,-6,0,0" 
                            Style="{StaticResource PhoneTextSubtleStyle}"/&gt;
             &lt;/StackPanel&gt;
         &lt;/StackPanel&gt;
&lt;/DataTemplate&gt;
</pre>

This DataTemplate is then used in the ListBox ItemTemplate by using the FeedItemTemplate name:

<pre class="brush: xml; title: ; notranslate" title="">&lt;ListBox x:Name="NewsLbx"
         Margin="0,0,-12,0"
         ItemTemplate="{StaticResource FeedItemTemplate}"
         ItemsSource="{Binding NewsItems}"
         Tap="NewsLbx_Tap"/&gt;
</pre>

This is more or less the only thing required for the view to work. Remember the complete source is available at Codeplex, link at the top of this post.

##  The ViewModel

Now if you&#8217;re wondering where the view is actually getting it&#8217;s data from, then read on!

In the project there is class, MainViewModel, this class is responsible of holding the data that the view uses and in this class has the ObservableCollections that contains the data.

The view is bound to the data in the App.xaml.cs:

{{< highlight  csharp >}}
	
		private static MainViewModel viewModel = null;

        public static MainViewModel ViewModel
        {
            get
            {
                // Delay creation of the view model until necessary
                if (viewModel == null)
                    viewModel = new MainViewModel();

                return viewModel;
            }
        }

{{< /highlight >}}

The ItemViewModel is pretty straight forward; the class is implementing the INotifyPropertyChanged which makes the View auto updated when the data in the model is changed. To implement this the following is done:

{{< highlight  csharp >}}
	
	public class FeedItemViewModel : INotifyPropertyChanged
	{
        private string title;
        public string Title
        {
            get { return title; }
            set
            {
                if (value != title)
                {
                    title = value;
                    NotifyPropertyChanged("Title");
                }
            }
        }

        ...
	}

{{< /highlight >}}

The most noticeable thing in the above is the call to NotifyPropertyChanged(&#8220;Title&#8221;) &#8211; this is to make the the view aware that a property has changed.

##  The RSS Consumer

To retrieve the  RSS data a WebClient is used:

{{< highlight  csharp >}}
	
	WebClient web = new WebClient();
    web.DownloadStringCompleted += new DownloadStringCompletedEventHandler(WebClient_DownloadStringCompleted);
    web.DownloadStringAsync(uri);
{{< /highlight >}}


When the download is completed the following are executed:

{{< highlight  csharp >}}
	
	StringReader sr = new StringReader(e.Result);
    var reader = XmlReader.Create(sr);
    var document = XDocument.Load(reader);

    var entries =
                 from entry in document.Descendants("channel").Descendants("item")
                 let title = entry.Element("title").Value
                 let link = entry.Element("comments").Value
                 let pubDate = entry.Element("pubDate").Value
                 select new FeedItemViewModel()
                 {
                     Title = title,
                     Url = link,
                     PubDate = DateTime.Parse(pubDate)
                 };
    feedHandler(entries.ToArray());
{{< /highlight >}}


By using LINQ the response from the RSS feed is handled and parsed into the FeedItemViewModel. This could probably be made a hole lot prettier, but hey; it&#8217;s open source and yours to edit on Codeplex.

That&#8217;s pretty much it!

If you have any comments/suggestions/questions, then please contact me :)



<div style="font-size:0px;height:0px;line-height:0px;margin:0;padding:0;clear:both">
</div>