---
layout: post
title: Coding the Hypermedia Elements of a Message
date: 2014-03-25 02:40 UTC
description: Writing code around hypermedia elements
---

I recently wrote about the idea of [converting from one hypermedia type to another](http://smizell.com/weblog/2014/converting-between-hypermedia-types), and how this was one goal of the [Hyperdescribe](https://github.com/smizell/hyperdescribe/) project I'm working on. I'd like to look at another idea, which involves writing API code around hypermedia elements rather writing code for specific hypermedia formats.

This is not too far from what we developers are already doing. We take our models, the properties of those models, add in some links, and generate a message in a hypermedia type of our choosing. What I'm proposing here is that we describe those entities, properties, and links in a standard format rather than a media type-specific format.

In this article, I'm going to look at the common hypermedia elements, take a brief look at a few popular hypermedia types, and then hit on how this can be useful.

## Common Hypermedia Elements

This section will look at the most common elements in a hypermedia message. After this, I'll hit on a few hypermedia formats to see how these elements fit in.

### Entities

Simply put, in hypermedia, an entity is anything that has identity. It can be something physical, like a car, or more abstract, like love. It can be real or it can be fictional. Some will refer to an entity as a resource or and others will call it a thing, and both of those bring with it the idea of identity.

When creating a response from a hypermedia API, the server is returning some entity or collection of entities, whether it be a customer or a collection of customers. When your browser requested this page you are reading, it requested a hypermedia document that consists of a blog post. It told my server, "I want the blog post resource at this URL and I understand the **text/html** hypermedia format." My server responded with the HTML representation of this entity, which you are probably viewing right now.

### Entity Properties

Things that have identity also have properties. A customer will have a name, address, phone number, and so on. This blog post for instance has a title, published date, and body. When we build databases, (most of the time) we create a table for our entities and then columns with data about those entity. We do the same thing on the web with resources.

### Hypertext Links and Controls

You'd be hard-pressed to find an API that does not have entities and properties, but not every API has links or controls. These links and controls are a key part to the hypermedia message, and are key elements to how the web works.

In a hypermedia message, there are links that tell the client what other entities it can request. These links usually have a link relation that tells the client what kind of entity will be found when that link is requested.

Additionally, some hypermedia messages have controls that tell the client what actions it can take while providing hints on how to take those actions. In HTML, for instance, you'd see these controls as `<form>` elements. These forms provide a template for building requests, and inform the client where to send the request along with what format to send it in.

For a much more granular look at the factors around hypermedia links and controls, please look at Mike Amundsen's document on [H Factors](http://amundsen.com/hypermedia/hfactor/).

## Examples

To get an idea for how these categories show up in hypermedia messages, let's have a look at some examples.

### HTML

Of course, HTML is a great example of a hypermedia type, which makes this page a good example message. As mentioned, this page is a blog post with a title, published date, and body text. It also has some extra, embedded entities, such as a collection of comments and individual comments themselves. It provides links to other entities on this site, along with links to other places on the web.

### HAL+JSON

Have a look at the JSON example in the [HAL spec](http://stateless.co/hal_specification.html). This particular message is a collection of orders. We have some properties for this collection of orders, `currentlyProcessing` and `shippedToday`, along with embedded individual order entities in the `_embedded` object. HAL has another reserved property name `_links` for specifying hypermedia links, and you can see how that is used for the collection and for the embedded items.

### Collection+JSON

Now check out the example from the [Collection+JSON example page](http://amundsen.com/media-types/collection/examples/).

Like in the HAL example, this message is a collection of items. The collection entity has embedded entities that are friends, and there are links sprinkled throughout the document. Additionally, there is a control template used for adding items to this collection (with a POST request, according to the spec) and for updating an item in the list (with a PUT request).

### Siren

Finally, check out the [Siren spec](https://github.com/kevinswiber/siren) for my last example. I won't spend too much time here because the elements are so obvious. You'll see entities, properties, links, and actions by name in the sample message.

## How is This Helpful?

With this common thread running through all media types, there is a great opportunity to come up with a way of abstracting out these core elements to be used across servers and clients.

On the server side, it allows us to build generic representers, which can then be converted to other media types. [ROAR](https://github.com/apotonick/roar) aims at doing this, but uses Ruby classes and modules rather than a document format.

On the client side, a generic client can be built that is able to understand several different media formats. There could be a method to "follow a link," and the client would be able to do this in HAL+JSON, Collection+JSON, and so on.

## In Conclusion

I understand how this may seem like I'm trying to [add a new standard](https://xkcd.com/927/) the list of hypermedia types. With Hyperdescribe, though, I am not proposing a new, better media type, but rather a way to describe the elements of the already-existing media types. I don't desire developers to use Hyperdescribe instead of their favorite media type, but rather to use Hyperdescribe to augment how they interface with their media type. Doing so will allow us to build libraries for the servers and clients that will let us think about elements rather than specific media types.
