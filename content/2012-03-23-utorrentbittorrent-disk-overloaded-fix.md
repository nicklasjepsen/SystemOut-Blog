---
title: "uTorrent/BitTorrent Disk Overloaded Fix"
author: "Nicklas Møller Jepsen"
date: "2012-03-23"
url: "/2012/03/23/utorrentbittorrent-disk-overloaded-fix/"
comments: "true"
categories:
  - off-topic
tags:
  - BitTorrent
  - Disk cache
  - Disk Overloaded
  - uTorrent
series:
---
This post is definitely not about programming, but I wasn’t able to find a fix for the uTorrent Disk Overloaded problem and decided to make a post about the solution.

<!--more-->

But first the problem: When downloading torrents using uTorrent/BitTorrent I have recently experienced a constant Disk Overloaded error displaying in the status bar causing any torrents to download at about max. 400-500 kb/sec. This is definitely not very nice if you’re downloading the newest Windows 8 beta’s or other large files.

The problem is obviously caused by uTorrent trying to write extremely often to the disk and therefore it just took 2 changes to the settings before my download speed accelerated again.

# 

#  Uncheck the write out settings

There are some post suggesting to uncheck the ‘Enable caching of disk writes’ but that just seems wrong, because we DO want to cache the writes so we’re not writing to the disk constantly. However the 2 sub settings we do want to uncheck; ‘Write out untouched blocks every 2 minutes’ and ‘ Write out finished pieces immediately’. Especially the last one is important.

So here’s my settings:

[<img style="background-image: none; border-bottom: 0px; border-left: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="uTorrent" border="0" alt="uTorrent" src="http://systemout.net/wp-content/uploads/2012/03/uTorrent_thumb.png" width="669" height="498" />][1]&nbsp;

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- links1 -->
<ins class="adsbygoogle"
     style="display:inline-block;width:728px;height:15px"
     data-ad-client="ca-pub-5807169669170468"
     data-ad-slot="6444119354"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

For the record; I also changed the cache size to 1800 and disabled Windows caching of disk reads/writes – not sure if that has any effect.



<div style="font-size:0px;height:0px;line-height:0px;margin:0;padding:0;clear:both">
</div>

 [1]: http://systemout.net/wp-content/uploads/2012/03/uTorrent.png