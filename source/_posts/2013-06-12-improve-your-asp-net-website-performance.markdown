---
layout: post
title: "Improve Your ASP .NET Website Performance"
date: 2013-06-12 11:48
comments: false
categories: ASP.NET, Performance
---

Improve Your ASP .NET Website Performance

# 5 minutes to better performance on your ASP .NET Web Site

Step 1:

Turn on Gzip.

How do you turn on gzip?

For most sites you want to put this into your web.config

 
	<system.webserver>
		<httpCompression directory="%TEMP%\iisexpress\IIS Temporary Compressed Files">
            <scheme name="gzip" dll="%IIS_BIN%\gzip.dll" />
            <dynamicTypes>
                <add mimeType="text/*" enabled="true" />
                <add mimeType="message/*" enabled="true" />
                <add mimeType="application/x-javascript" enabled="true" />
                <add mimeType="*/*" enabled="false" />
            </dynamicTypes>
            <staticTypes>
                <add mimeType="text/*" enabled="true" />
                <add mimeType="message/*" enabled="true" />
                <add mimeType="application/x-javascript" enabled="true" />
                <add mimeType="application/atom+xml" enabled="true" />
                <add mimeType="application/xaml+xml" enabled="true" />
                <add mimeType="*/*" enabled="false" />
            </staticTypes>
        </httpCompression>
        <urlCompression doStaticCompression="true" doDynamicCompression="true"/>
    </system.webserver>
 
You may need to enable Compression in IIS if you are running an older version.

Step 2:

Set the correct cache headers on your javascript and css

 
	<system.webserver>
            <staticContent>
               <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="350.00:00:00" />
            </staticContent>
	</system.webserver>
 
This says cache all my static content for 350 days

**Warning** You may not want to do this if you aren't doing some kind of cache invalidation with your file names.

Step 3:

Minify and Combine all of your javascript and css into as few requests as possible per page

Minify - Strip all white space and comments from your javascript and css
Combine - Put as many files into one file as you can

Easiest way to do this:

Install Request Reduce in each project you want to combine and minify in

www.requestreduce.org

Nuget Action:

    Install-Package RequestReduce

You shouldn't have to config anything and it should work out of the box

Slightly more complicated ways of doing this

Use a bundler

the ASP .NET bundler is great

Cassette  is also awesome

BAM!

You've now just probably sped your page up!