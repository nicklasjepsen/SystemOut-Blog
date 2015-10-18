---
title: "Publish WCF Service on IIS"
author: "Nicklas Møller Jepsen"
date: "2012-03-20"
url: "/2012/03/20/publish-wcf-service-on-iis/"
comments: "true"
categories:
  - programming
  - deploying
tags:
  - wcf
  - iis
  - Binding
  - Hostname
  - Publishing schema
  - WSDL
series:
---
Recently I needed to publish a WCF service on an IIS. This progress I quite smooth when you just need to access the WCF service locally, but ones you need to publish the service on the great Internet, things are a little more difficult.  
<!--more-->

#  Publishing the WSDL

The ‘WSDL generation strategy’ when using WCF is that you get multiple documents to describe your service. This can lead to some problems if you’re just publishing without making some customizations.

Have a look at the wsdl:types:

<pre class="brush: xml; title: ; notranslate" title="">&lt;wsdl:types&gt;
    &lt;xsd:schema targetNamespace="http://tempuri.org/Imports"&gt;
        &lt;xsd:import schemaLocation="http://localhost/Service/Service.svc?xsd=xsd0"namespace="http://tempuri.org/"/&gt;
        &lt;xsd:import schemaLocation="http://localhost/Service/Service.svc?xsd=xsd2"namespace="http://schemas.datacontract.org/2004/07/Service.Interface"/&gt;
    &lt;/xsd:schema&gt;
&lt;/wsdl:types&gt;
</pre>

Notice the localhost, that will definitely cause some trouble if you need to consume the WSDL from an external machine.

#  How to fix this?

There’re two ways to solve this (if not more):

*   Fix the host name (this post)
*   Generate a single WSDL (future post)

##  Fix the host name

It’s actually the IIS who are responsible of the host name. Therefore we need to configure the bindings for the web site the WCF service is hosted on.

Go to IIS Manager, right click the web site and choose Edit bindings:

[<img style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border: 0px;" title="iis_tree" src="http://systemout.net/wp-content/uploads/2012/03/iis_tree_thumb.png" alt="iis_tree" width="211" height="74" border="0" />][1]

Now click the Add button and enter the external host name or ip where your service/wsdl will be available from:

[<img style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border: 0px;" title="addbinding" src="http://systemout.net/wp-content/uploads/2012/03/addbinding_thumb.png" alt="addbinding" width="244" height="133" border="0" />][2]

Restart the web site and go to <http://mydomain.com/serviceAlias/Service.svc?wsdl> and your should see that the imports are now using the external host name.

##  Generate a single WSDL

You can also choose to generate a single WSDL file for your WCF service. I will post about that later, meanwhile take a look at <a href="http://weblogs.asp.net/pglavich/archive/2010/03/16/making-wcf-output-a-single-wsdl-file-for-interop-purposes.aspx" target="_blank">this great article</a> that describes it – although I find it a bit to complex and therefore are going to write my own post about it when the time comes <img class="wlEmoticon wlEmoticon-smile" style="border-style: none;" src="http://systemout.net/wp-content/uploads/2012/03/wlEmoticon-smile.png" alt="Smile" />



<div style="font-size:0px;height:0px;line-height:0px;margin:0;padding:0;clear:both">
</div>

 [1]: http://systemout.net/wp-content/uploads/2012/03/iis_tree.png
 [2]: http://systemout.net/wp-content/uploads/2012/03/addbinding.png