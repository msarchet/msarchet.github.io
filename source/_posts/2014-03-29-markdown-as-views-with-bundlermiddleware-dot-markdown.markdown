---
layout: post
title: "Markdown As Views with BundlerMiddleware.Markdown"
date: 2014-03-29 11:43
comments: true
categories: OWIN, Middleware, Markdown, Bundler, BundlerMiddleware
---

# BundlerMiddleware.Markdown

See my previous [post](http://msarchet.com/using-the-asp-dot-net-bundling-pipeline-with-owin/) where I talk about using the `System.Web.Optimization` pipeline in an OWIN middleware to simplify your single page apps. To continue on that idea I have created the `BundlerMiddleware.Markdown` package for transforming Markdown files into HTML.

## Installation

`Install-Package BundlerMiddleware.Markdown`

## Usage

Using the markdown middleware is as simple as using the normal `BundlerMiddleware`. First you need to set up some routes that point at your Markdown files. Then you need to include the middleware in your OWIN startup class.

### Routes

One thing to note is that in this version BundlerMiddleware and BundlerMiddleware.Markdown need seperate route tables to run side by side without route conflicts.

> While you could use the same `BundlerRouteTable` it will cause anything to be served as Markdown or HTML. I'm working on some API cleanups to make this feasible. For now I suggest making a static `BundlerRouteTable` in your config file

Add some routes to the `BundlerRouteTable`

	// create a static table for the markdown routes
    public static BundlerRouteTable MarkdownRoutes = new BundlerRouteTable();

    // add a route to a markdown file
    MarkdownRoutes.FromVirtualPath("~/docs/doc1.md", "/docs/doc1.md");

### Startup

Then wire up in your `Startup.cs`

	app.UseBundlerMarkdown(MarkdownRoutes);

### Templating

If you want to apply some global styles to your markdown files you can create a template file that has a

    !!block content!!

inside the body tag of the document. This is where the markdown content will be injected.

A completed file could look like

    <doctype html>
    <head>
	    <title>Markdown</tile>
	    <!-- using the bundling is allowed as well -->
	    !!styles:~/styles/main!!
    </head>
    <body>
		!!block content!!
		!!scripts:~/bundles/main!!
    </body>
    </html>

To make use of the template file, instead of doing `app.UseBundlerMarkdown` you should use `app.UseBundlerMarkdownWithTemplate(MarkdownRoutes, "~/template.html");`.

## Considerations

BundlerMiddleware and BundlerMiddleware.Markdown are still not a 1.x release so do expect API changes and potential bugs. However I do appreciate pull requests or issues on the [github repo](http://github.com/msarchet/bundler).

A shorter form of this is available on the project page at [msarchet.com/Bundler](http://msarchet.com/Bundler)
