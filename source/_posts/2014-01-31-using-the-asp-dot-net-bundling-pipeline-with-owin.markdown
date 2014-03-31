---
layout: post
title: "Using the ASP.NET Bundling Pipeline with OWIN"
date: 2014-01-31 08:00
comments: true
categories: [OWIN, ASP, .NET, C#, Programming, Bundling, Middleware, Optimization, Single Page Applications, SPA, Angular]
---

# Using the ASP.NET Bundling Pipeline with OWIN

> To start off this is still running on top of IIS, I haven't fully done the work to make an extensible version that runs without `System.Web`.

## Why

There is has been a large shift to building client heavy web applications, or Single Page Applications, using frameworks like [Angular](angularjs.org). With this shift a large number of developers have begun exposing API's and building their applications by consuming those API's. However there is still a trend of using MVC frameworks to deliver what should effecitvely be static HTML to the browser.

Currently bundling support in ASP.NET relies on the `System.Web.Optimization` package, not on MVC or Razor as you would think based off how the tool is presented. However since MVC is the normal to serve HTML from an ASP.NET application a lot of people will serve their base page for their application as a *.cshtml file off of a MVC Controller. This requires setting up the routing, a controller, and view to get out what is effecitvely static HTML.

[Owin](http://owin.org) is now part of the defacto pipeline for handling requests in ASP .NET applications, so shouldn't we be able to hook in when a request comes and inject our bundles and return the static HTML back to the consumer.

## What

What if you set your bundling up as normal, added a route pointing to the files you wanted to use the bundling on, and the application served back the HTML.

What would this look like :

### /App/Views/index.html

	<html>
		<head>
			<title> Some Page </title>
			!!styles:~bundle/styles!!
		</head>
		<body>
			<div>Things!</div>
			!!scripts:~bundles/scripts!!
		</body>
	</html>
	
### global.asax.cs

Next we need to add the routes and the files that we want to transform and serve for those routes. To do this we add new `BundlerRoute` to the static `BundlerRoutes.Routes` collection. A `BundlerRoute` is is constructed with a route and a file path. The `BundlerMiddleware.System.Web` package includes an extension method for adding routes that are resolved from a Virtual Path. 

	BundlerRoutes.Routes.FromVirtualPath("/", "~/App/Views/index.html"));
	
Alternatively you can use the pattern that Web API or the BundleCollection uses and pass the BunderRoutes.Routes into a static function to create the routes.
	
### Startup.cs

	app.UseBundlerMiddlewareForIIS();
	
This is an extension method for making use of the `DefaultFileResolver` and `DefaultBundleResolver`.
	
## What is actually happening

BundlerMiddleware checks each request to see if the request matches a registered route in its table. If the route matches it reads the file, calls into the System.Web.Optimization library to get out the appropriate tags for the bundle that it matched, writes the tags into the html and returns
	
	<html>
		<head>
			<title> Some Page </title>
			<link rel="stylesheet" src="bundles/main.min.css" />
		</head>
		<body>
			<div>Things!</div>
			<script src="bundles/scripts.js?v=<hash>" type="text/javascript" />
		</body>
	</html>

Or injects appropriately the debug tags if you are running in debug mode

## How can you use it

You can use this by installing the nuget package

	BundlerMiddleware.System.Web

This provides the necessary code to use BundlerMiddleware with the standard `System.Web.Optimization` bundling and minification hooks

## What if you don't want to use System.Web

If you don't want to or can't use `System.Web` you should just install `BundlerMiddleware`. When using the base library you will need to create your own implementation of `IFileResolver` and `IBundleResolver`.

Example without implementation

	public class CustomFileResolver : IFileResolver {}
	
	public class CustomBundleResolver : IBundleResolver {} 

then in your `Startup.cs` you would just do

	app.Use(typeof(BundlerMiddleware), new CustomFileResolver(), new CustomBundleResolver())
	


