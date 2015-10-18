---
title: "'Code Snippet &#8211; &#8216;Default&#8217; regions in C# class'"
author: "Nicklas Møller Jepsen"
date: "2011-12-20"
url: "/2011/12/20/code-snippet-default-regions-in-c-class/"
comments: "true"
categories:
  - programming
tags:
  - c-sharp
  - code-snippet
  - Code standards
  - Regions
  - Snippet
  - visualstudio
  - VS
series:
---
I&#8217;m currently trying to standardizes the way I&#8217;m writing code. To achieve this I simply try to comply to some simple code standards. This is, in some areas, easy to accomplish because of the Visual Studio auto format option.<!--more-->

###  Why use Regions in C

The auto format of source code, however, do not take care off where in the file the different parts that together becomes a class is placed. Therefore I decided to use regions. Regions are good at many things for instance you can collapse/expand your regions (see below) and you can group methods/properties/fields and so forth accordingly.

Expanded:

[<img class="alignnone size-full wp-image-77" style="border-color: initial; border-style: initial;" title="Expanded" src="http://systemout.net/wp-content/uploads/2011/12/expanded.png" alt="" width="207" height="116" />][1]

Collapsed:

<img class="alignnone size-full wp-image-78" title="Collapsed" src="http://systemout.net/wp-content/uploads/2011/12/collapsed.png" alt="" width="157" height="85" />

###  The C# Code Snippet

Well, now to the Code Snippet as I&#8217;ve created based on this <a href="http://stackoverflow.com/questions/603758/whats-the-best-way-to-layout-a-c-sharp-class" target="_blank">StackOverflow </a>post.

The snippet creates this layout of the class:

<img class="alignnone size-full wp-image-81" title="SnippetCollapsed" src="http://systemout.net/wp-content/uploads/2011/12/snippetcollapsed.png" alt="" width="187" height="219" />

Of course a class can contain a lot more then this, but I think as a general purpose snippet, this will do.

The XML of the snippet is here:

<pre class="brush: xml; title: ; notranslate" title="">&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;CodeSnippets xmlns="http://schemas.microsoft.com/VisualStudio/2005/CodeSnippet"&gt;
	&lt;CodeSnippet Format="1.0.0"&gt;
		&lt;Header&gt;
			&lt;Title&gt;Standard Class&lt;/Title&gt;
			&lt;Shortcut&gt;stdcl&lt;/Shortcut&gt;
			&lt;Description&gt;Create standard C# regions for this class.&lt;/Description&gt;
			&lt;Author&gt;Nicklas Møller Jepsen&lt;/Author&gt;
			&lt;SnippetTypes&gt;
				&lt;SnippetType&gt;Expansion&lt;/SnippetType&gt;
			&lt;/SnippetTypes&gt;
		&lt;/Header&gt;
		&lt;Snippet&gt;
		&lt;Declarations&gt;
			&lt;Literal default="true" Editable="false"&gt;
                    &lt;ID&gt;classname&lt;/ID&gt;
                    &lt;ToolTip&gt;Class name&lt;/ToolTip&gt;
                    &lt;Function&gt;ClassName()&lt;/Function&gt;
                    &lt;Default&gt;ClassNamePlaceholder&lt;/Default&gt;
                &lt;/Literal&gt;

		&lt;/Declarations&gt;
		&lt;Code Language="csharp" Kind="method head"&gt;&lt;![CDATA[#region Fields

				#endregion

				#region Constructors

				public $classname$() { }

				#endregion

				#region Properties

				#endregion

				#region Methods

				#endregion
		$end$
		]]&gt;
		&lt;/Code&gt;
		&lt;/Snippet&gt;
	&lt;/CodeSnippet&gt;
&lt;/CodeSnippets&gt;
</pre>

You can download the snippet <a href="http://dl.dropbox.com/u/212154/systemoutdotnet/stdcl.snippet" target="_blank">here</a>.

Please let me know if you have any comments to the snippet or let me know if you have any great ideas for some new snippets :) 

&nbsp;

Cheers, Nicklas

&nbsp;



<div style="font-size:0px;height:0px;line-height:0px;margin:0;padding:0;clear:both">
</div>

 [1]: http://systemout.net/wp-content/uploads/2011/12/expanded.png