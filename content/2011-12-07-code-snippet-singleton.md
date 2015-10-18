---
title: "'Code Snippet &#8211; Singleton'"
author: "Nicklas Møller Jepsen"
date: "2011-12-07"
url: "/2011/12/07/code-snippet-singleton/"
comments: "true"
categories:
  - programming
tags:
  - code-snippet
  - c-sharp
  - Design patterns
  - Singleton
  - Snippet
series:
---
I&#8217;ve written some code snippets which makes the daily job as a developer more convenient. These snippets are for use with Visual Studio.

If you want to have a look at how these snippets are added to Visual Studio see how to add a snippet [here.][1]

This post is about the Singleton pattern and the snippet I use to quickly implement this pattern in a class.

<!--more-->

The <a title="Singleton" href="http://en.wikipedia.org/wiki/Singleton_pattern" target="_blank">Singleton pattern</a> is a pattern that make sure that only one instance of an object exists. Al thou there might be pros and cons for using the pattern, but for instance settings classes and controllers it probably is best that these are always single instances.

The snippet looks like this:

<pre class="brush: xml; title: ; notranslate" title="">&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot;?&gt;
&lt;CodeSnippets xmlns=&quot;http://schemas.microsoft.com/VisualStudio/2005/CodeSnippet&quot;&gt;
	&lt;CodeSnippet Format=&quot;1.0.0&quot;&gt;
		&lt;Header&gt;
			&lt;Title&gt;Singleton&lt;/Title&gt;
			&lt;Shortcut&gt;sglt&lt;/Shortcut&gt;
			&lt;Description&gt;Implement the Singleton pattern here.&lt;/Description&gt;
			&lt;Author&gt;Nicklas Møller Jepsen&lt;/Author&gt;
			&lt;SnippetTypes&gt;
				&lt;SnippetType&gt;Expansion&lt;/SnippetType&gt;
			&lt;/SnippetTypes&gt;
		&lt;/Header&gt;
		&lt;Snippet&gt;
		&lt;Declarations&gt;
			&lt;Literal&gt;
				&lt;ID&gt;instancepropname&lt;/ID&gt;
				&lt;ToolTip&gt;The name of the property to get a Singleton instance&lt;/ToolTip&gt;
				&lt;Default&gt;Instance&lt;/Default&gt;
			&lt;/Literal&gt;
			&lt;Literal default=&quot;true&quot; Editable=&quot;false&quot;&gt;
                    &lt;ID&gt;classname&lt;/ID&gt;
                    &lt;ToolTip&gt;Class name&lt;/ToolTip&gt;
                    &lt;Function&gt;ClassName()&lt;/Function&gt;
                    &lt;Default&gt;ClassNamePlaceholder&lt;/Default&gt;
                &lt;/Literal&gt;

		&lt;/Declarations&gt;
		&lt;Code Language=&quot;csharp&quot; Kind=&quot;method head&quot;&gt;&lt;![CDATA[private static $classname$ _instance;
		private $classname$() {}

		public static $classname$ $instancepropname$
		{
			get
			{
				if (_instance == null)
					_instance = new $classname$();
				return _instance;
			}
		}
		$end$
		]]&gt;
		&lt;/Code&gt;
		&lt;/Snippet&gt;
	&lt;/CodeSnippet&gt;
&lt;/CodeSnippets&gt;
</pre>

Remember to use the Source, Luke &#8211; so don&#8217;t go abusing the above. Singletons are good, but only when they are needed. Or something like that :) 



<div style="font-size:0px;height:0px;line-height:0px;margin:0;padding:0;clear:both">
</div>

 [1]: http://systemout.net/blog/2011/12/07/code-snippet-how-to-add-visual-studio/ "Code Snippet – How to add to Visual Studio"