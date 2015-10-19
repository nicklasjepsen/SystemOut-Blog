---
title: "'C# Generic Xml Parser Example'"
author: "Nicklas Møller Jepsen"
date: "2013-12-19"
url: "/2013/12/19/csharp-generic-xml-parser-example/"
comments: "true"
weight: 500
categories:
  - programming
tags:
  - c-sharp
  - avoid duplicate code
  - api
  - generic
  - sample code
  - xml
series:
---
In this post I will show how to implement and use a generic xml parser library.

For the complete solution and source code, <a href="http://sdrv.ms/18BwjwK" title="XmlParser.zip" target="_blank">click here!</a>

I wanted to add a tool to my source toolbox, a library to use when handling xml data representations because I was tired of writing/googling the same implementations each time I had the need for some xml parsing.  
<!--more-->

  
Therefore I wrote a tiny simple XmlParser which can be invoked like so:

<pre class="brush: csharp; title: ; notranslate" title="">// Create some persons and get their XML represantation
            var me = new Person
            {
                Name = "Nicklas Møller Jepsen",
                YearBorn = 1985,
                Address = new Address
                {
                    StreetName = "Mainstreet",
                    StreetNumber = 10,
                    City = "Copenhagen",
                    PostalCode = "2100",
                }
            };

            var myGirlfriend = new Person
            {
                Name = "Scarlett Johansson",
                YearBorn = 1984,
                Address = me.Address // Of course!
            };

            // Now get the xml from the objects!
            var meXml = XmlParser.ToXml(me);
            var girlfriendXml = XmlParser.ToXml(myGirlfriend);
</pre>

&#8230;and to get the object representation from the xml string simply do the following:

<pre class="brush: csharp; title: ; notranslate" title="">// Now parse the xml into a new object
            var newMe = XmlParser.ToObject&lt;Person&gt;(meXml);
</pre>

Quite simple!  
The source code for the parser is not more than a few lines of code and wrapped in some exception handling so I only need to worry about ONE exception, namely the XmlParserException.

<pre class="brush: csharp; title: ; notranslate" title="">public static string ToXml&lt;T&gt;(T objectToParse) where T : class, new()
        {
            if (objectToParse == null)
                throw new XmlParserException("Unable to parse a object which is null.", new ArgumentNullException("objectToParse"));

            var stringwriter = new System.IO.StringWriter();
            var serializer = new XmlSerializer(typeof(T));
            try
            {
                serializer.Serialize(stringwriter, objectToParse);
            }
            catch (Exception e)
            {
                throw new XmlParserException(string.Format("Unable to serialize the object {0}.", objectToParse.GetType()), e);
            }

            return stringwriter.ToString();
        }

        public static T ToObject&lt;T&gt;(string xmlTextToParse) where T : class, new()
        {
            if (string.IsNullOrEmpty(xmlTextToParse))
                throw new XmlParserException("Invalid string input. Cannot parse an empty or null string.", new ArgumentException("xmlTestToParse"));

            var stringReader = new System.IO.StringReader(xmlTextToParse);
            var serializer = new XmlSerializer(typeof(T));
            try
            {
                return serializer.Deserialize(stringReader) as T;
            }
            catch (Exception e)
            {
                throw new XmlParserException(string.Format("Unable to convert to given string into the type {0}. See inner exception for details.", typeof(T)), e);
            }
        }
</pre>

Nothing special going on here, using some XmlSerializers &#8216;n stuff. But the main value of this is that it is wrapped up in my toolbox library so I can just reference this class whenever I need it.

Let me know what you think and if you have any ideas what else should be in a developer source toolbox and I will add it!

For the complete solution and source code, <a href="http://sdrv.ms/18BwjwK" title="XmlParser.zip" target="_blank">click here!</a>



<div style="font-size:0px;height:0px;line-height:0px;margin:0;padding:0;clear:both">
</div>