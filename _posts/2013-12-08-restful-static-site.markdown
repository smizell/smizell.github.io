---
layout: post
title: RESTful Static Site
date: 2013-12-08 00:30 UTC
category: technology
tags: rest, static
description: Is your API more RESTful than a static HTML site?
---

In modern development, the terms REST and API have become quite ambiguous. In web development, an API is usually a means of exposing your data over HTTP, while REST means your API has some basic qualities and characteristics, mainly using HTTP verbs to perform CRUD operations.

As I've been reading about REST, I see that REST is a much more robust topic than this, and covers a wider topic than access to data. I was was thinking through this while putting up this little static site and it got me to thinking: **is this tiny, static HTML site more RESTful than a lot of HTTP APIs out there**?

## Compared to the Most Common View of REST

As I mentioned, a majority of modern developers define REST much differently than the creator of REST would define it. Usually, REST takes on these characteristics for most developers.

1. **RESTful routes** - This pretty much means clean URLs. Many call these endpoints.
2. **HTTP verbs** - Instead putting your actions in the URL (like /get_something) you use the HTTP verbs to perform actions on resources, such as requesting a resource with a GET verb.
3. **CRUD** - The verbs are used to perform database-like actions on data, which are Create, Read, Update, and Delete. These translate to POST, GET, PUT/PATCH, and DELETE HTTP verbs.
4. **Multiple formats** - Adding .json or .xml to the file name will give you different formats of the data, or you can use the Accept header in HTTP to request a specific format.

How does this static site stack up to these concepts? First, this site has clean URLs. Even though this is an HTML site, I was able to leave off the file extension because of Apache's [content negotiation](http://httpd.apache.org/docs/2.2/content-negotiation.html) and link to the resource instead of a specific representation of it. More on this later.

Next, HTTP verbs can be used to get any resource on this site. Everything is a GET here on this site (as of the time of this writing), but that doesn't make it non-RESTful.

Additionally, CRUD is here, though I'm just using the read aspect of CRUD.

Lastly, because of content negotiation, I can provide different formats of a file (i.e. resource) at the same URI. I've put together an example for how this happens.

1. I've made an example resource located at [/examples/rest/test](/examples/rest/test).
2. When you send a GET request for that resource to my server, your browser tells my server what content types it understands.
3. The server takes that information and looks for the best representation of the resource for the client (your browser).
4. Because your client prefers HTML, the server will give you the HTML representation, which is the file [/examples/rest/test.html](/examples/rest/test.html).
5. If you had a client that could only understand JSON, the server would return the JSON representation of that resource at [/examples/rest/test.json](/examples/rest/test.json). You could test this out with curl and put "application/json" in the Accept header.

If you are Rails developer, this will look very familiar to what Rails does with the [respond_to](http://apidock.com/rails/ActionController/MimeResponds/InstanceMethods/respond_to) block. I'm doing the same thing here, it's just with Apache and static files.

## Compared to True REST

REST is an architecture that was created and defined by Roy Fielding, and comes with a completely different way of thinking that simply accessing data with CRUD operations. The architecture has several constraints that must be applied to truly make something RESTful. One constraint commonly neglected is the Uniform Interface, which has a concept with it called Hyptertext as the Engine of Application State ([HATEOAS](http://en.wikipedia.org/wiki/HATEOAS)). This basically means the server includes hypertext links and forms that drive the client on what it can do with a given resource.

This is very easy to see accomplished with HTML—it's how the web currently works. The responses the server gives to your client, the browser, include links and forms that allow you to follow your nose toward other resources. This site does this very thing with HTML and holds to the HATEOAS constraint.

## Bonus Round

I was looking through the [slides of a recent talk by Roy Fielding](http://www.slideshare.net/evolve_conference/201308-fielding-evolve), and one of them said this below. Fielding is the one who first defined REST.

> A REST API is just a website for users with a limited vocabulary (machine to machine interaction)

In REST over HTTP, you use content types to communicate the format of data between machines. There are things like microdata, microformats, and RDF that allow you to specify vocabulary for your data within your media types. I've attempted to do that on this site with [Schema.org](http://schema.org/) vocabulary, which you can explore by viewing the source. This allows me to use a vocabulary that can be understand by a client so that the client can understand the resource.

## In Closing

As you can see, a static HTML site can actually be extremely RESTful, and is actually a really good way to approach thinking about REST. With a static server, you have resources (a blog post such as this), representations of that blog post (you are probably viewing the HTML representation), and hypermedia (HTML is a hypermedia type). This site also uses a certain vocabulary for marking up the data.

There are also other things I didn't get into here. Apache will handle [Etags](http://en.wikipedia.org/wiki/HTTP_ETag) right out of the box. It will do language negotiation and charset negotiation as well. Apache has a lot of great functionality for handling web requests in a RESTful way.

So back to my original question, I believe that static sites like this can actually be very RESTful and holds it own against APIs out there claiming to be RESTful.
