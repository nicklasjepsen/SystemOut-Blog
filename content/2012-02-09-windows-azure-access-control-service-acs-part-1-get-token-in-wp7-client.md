---
title: "'Windows Azure Access Control Service (ACS) &#8211; part 1 &#8216;Get token in WP7 client&#8217;'"
author: "Nicklas Møller Jepsen"
date: "2012-02-09"
url: "/2012/02/09/windows-azure-access-control-service-acs-part-1-get-token-in-wp7-client/"
comments: "true"
categories:
  - programming
tags:
  - c-sharp
  - security
  - wp7
  - Windows Azure
  - accesscontrol
  - acs
  - authorization header
  - REST
  - RestSharp
  - WCF
  - windows phone
series:
---
While working on a project for WP7 where a backend is required I needed to implement some sort of security on the server. As the server is being hosted in Windows Azure I looked into ACS. At first it did look a bid &#8220;over configured&#8221; meaning there is a lot of documentation on how to use it and what you can actually do with it. Facebook, Google, Windows Live login etc. For my purpose I just needed a simple way off securing my REST service and after some reading I thought I would give ACS a try.

<!--more-->Steps involved:

*   Create a REST ACS enabled WCF service (will post about this subject later)
*   Configure ACS (Windows Azure Portal)
*   Request token on client (this post ;))

<div>
  This first post about ACS is about the last step in the above list. The reason for not starting from the top is that I couldn&#8217;t seem to find any decent guides on how to implement a <span style="text-decoration: underline;">simple</span> ACS client/consumer on Windows Phone. Therefore the bottom up concept :)
</div>

To actually configure ACS to Issue a SWT Token to be using I followed <a href="http://msdn.microsoft.com/en-us/library/hh289317.aspx" target="_blank">this well explained MSDN article</a> - scroll to the 2. part &#8216;Step 2 – Configure ACS to Issue a SWT Token&#8217;

Btw. in the examples below I&#8217;m using <a href="http://restsharp.org/" target="_blank">RestSharp</a> a very simple to use REST library for both WP7/SL, WinForms, etc.

And now back to the topic &#8211; WP7 ACS token consumer:

The basic concept of authorizing against a WCF ACS secured service is that you send an &#8220;Authorization&#8221; header in the request:

<pre class="brush: csharp; title: ; notranslate" title="">request.AddHeader("Authorization", token.TokenString);</pre>

But of course you need to now the token to put in the header.

To make sure we have the token, the following is executed before the actual REST call:

<pre class="brush: csharp; title: ; notranslate" title="">RestClient client = new RestClient(
string.Format("https://{0}.{1}", serviceNamespace, accesscontrol.windows.net));
RestRequest treq = new RestRequest("/WRAPv0.9");
treq.Method = Method.POST;
treq.AddParameter("wrap_name", uid);
treq.AddParameter("wrap_password", pwd);
treq.AddParameter("wrap_scope", realm);
client.ExecuteAsync(treq, (re) =&amp;gt;
{
// Get expiration
string expiry = result
.Split('&amp;amp;')
.Single(value =&amp;gt; value.StartsWith("wrap_access_token_expires_in", StringComparison.OrdinalIgnoreCase)).
Split('=')[1];

// Get Access Token
result = HttpUtility.UrlDecode(
result
.Split('&amp;amp;')
.Single(value =&amp;gt; value.StartsWith("wrap_access_token=", StringComparison.OrdinalIgnoreCase))
.Split('=')[1]);

token = new AcsToken()
{
ExpirationDate = DateTime.Now.AddSeconds(Int32.Parse(expiry)),
TokenString = string.Format("WRAP access_token="{0}"", result),
};
});
</pre>

Now the token is stored in a local variable &#8216;token&#8217; where it is possible to check if it is expired and to get the token string which the can be used to put in the header of coming requests against the ACS secured WCF REST service.

And just for as a ending note; here&#8217;s the Token type:

<pre class="brush: csharp; title: ; notranslate" title="">class AcsToken
{
public string TokenString { get; set; }
public DateTime ExpirationDate { get; set; }

public bool IsExpired
{
get
{
return DateTime.Now &amp;gt; ExpirationDate;
}
}
}
</pre>

That&#8217;s it for this part of the ACS post series. More will follow :)



<div style="font-size:0px;height:0px;line-height:0px;margin:0;padding:0;clear:both">
</div>