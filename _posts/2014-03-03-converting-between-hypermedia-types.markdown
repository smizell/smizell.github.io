---
layout: post
title: Converting Between Hypermedia Types
date: 2014-03-03 06:17 UTC
tags: rest, hypermedia, hyperdescribe
description: Supporting multiple hypermedia types by converting from one to another
---

Building a hypermedia API can be hard. Many times, we build our APIs around one specific hypermedia format, while either not supporting other formats or minimally supporting one or two. This article aims to look at an idea for building your API in one format, and then converting to other formats automatically.

I'd like to first look at our current API practices, and then explore this idea of converting between hypermedia formats. I have also started a project around this idea called [Hyperdescribe](https://github.com/smizell/hyperdescribe) that I'll look at toward the end as well.

## Current Design Practices

The majority of APIs these days do not include hypermedia in the responses, and seem to simply be model data that is serialized into JSON. There have been some great strides in hypermedia design to make going from simple, serialized JSON to hypermedia, though. You see this in formats like [JSON API](http://jsonapi.org/) and [HAL+JSON](http://stateless.co/hal_specification.html), which both try to reduce the friction of moving from basic JSON serialization to hypermedia. Even so, supporting multiple of these format is rarely done.

If the designers and developers of an API desire to include hypermedia from the beginning, they usually spend a great deal of time deciding which of these hypermedia format would be best to use, and then build an entire application that responds with that format almost exclusively. Other formats are supported in some cases, but the secondary representations are usually quite bare in functionality compared to the primary representation.

This becomes even more apparent when this happens for web applications, specifically Single Page Applications (SPA). In a previous article, I spent some time looking at [Discourse](http://www.discourse.org/) to show how they built a JSON API while also maintaining a very simple HTML version, which is served to crawlers and users without Javascript. The HTML version is minimal in what it provides, and requires to be maintained alongside their JSON representations.

## Thinking About Things Differently

I personally feel there are better ways to think about API design and development. One library that I feel gets closer to this is a Ruby library called [ROAR](https://github.com/apotonick/roar). ROAR allows you to define your resources with Ruby modules and then render that resource to either ROAR's custom format or to more common types like HAL or Collection+JSON.

With ROAR, instead of building to a specific representation, you describe your resource in a generalized way (with a representer) and convert from that to other hypermedia formats.

This seems to be getting closer to a better way to think about API design, but I feel it can be improved upon. ROAR describes resources and documents but does so with Ruby code. I think a better answer is to describe your resources in a format that is designed to describe hypermedia resources.

## Describing Hypermedia Messages

The idea here is that you design your API in such a way that you build one representation of your resource and convert from that to other resources.

As I explored in [my previous article](http://smizell.com/weblog/2014/html-hypermedia-api-decoupled-ui), there are a lot of benefits to be found in using HTML as your hypermedia format for your API. The common feeling is that this is an either/or choice, and that if you go with HTML and put all of your hard work into that format, you won't have a JSON format for other clients, or your JSON format will be lacking because it's too much work. It's hard enough to maintain one representation, and even more so for two.

If you could convert from HTML to a JSON hypermedia format like HAL automatically, though, you get the best of both worlds. Your API could respond with HTML or HAL when requested, and this could all be done by maintaining one hypermedia format.

To make this work, there has to be a description format that allows you to describe your resource in a flexible and extensive way, and then convert to and from that. If we build converters to and from every possible hypermedia format without this intermediary format, we'd end up with a hundred different converters out there to choose from and maintain. Using a standardized description format allows you convert to that format, and from there to any other supported format.

## Hyperdescribe Format

As I mentioned, I've started a project to see if there is any interest around this idea of having a description format. This project is called [Hyperdescribe](https://github.com/smizell/hyperdescribe), and it seeks to be a way to describe your hypermedia messages. It's goal is to be flexible while also covering as many hypermedia factors as possible.

![Hypermedia Diagram](https://github.com/smizell/hyperdescribe/raw/master/files/img/hyperdescribe.png)

Once you have a standardized way of describe a message, you can then have libraries that convert from and to that format.

## Uses for a Description Format

As I've outlined on the project page, there are several uses for a description format like this, some of which I've already hit upon as well.

**As I mentioned, you could build your API with one media type and then convert to other media types**. It would create a simpler environment for developers since they'd just maintain one format, and if you used HTML, you could gain all of its benefits while still fully supporting a JSON hypermedia format.

**Additionally, you could do the media type conversion in the browser (or client) instead of on the server**. This would allow a client that only understands HAL to work with a server that only responds with Siren. It reduces the friction between clients and servers.

**Of course, you could build your representer in Hyperdescribe and convert from there**. The goal of course is to provide a way to easily serve multiple media types from an API while also supporting multiple media types in the client.

## In Closing

In closing, the idea here is that instead of being stuck in one hypermedia format, you can build your API to one type and get the others for free. This allows for better APIs that serve many media types, along with better clients that understand more media types. I think in the end, we are left with a better web.
