---
layout: post
title: Extending Hypermedia with Hyperextend
date: 2014-05-15 02:04 UTC
tags: hypermedia hyperextend
description: Using Hyperextend to extend hypermedia formats
---

In the world of hypermedia, there are many opinions of what should or should not be in the responses of an API. Some designers consider resource semantics and actions to be out-of-band, so these elements are pushed to profiles and documentation. Others want these elements as part of the message, so we have media types that provide ways for including this information. In the midst of all of this, it's sometimes hard to decide what hypermedia format fits the job.

To help with this, I've created [Hyperextend](https://github.com/smizell/hyperextend), which is a library of hypermedia components that can be used for extending JSON hypermedia formats. It provides schemas for links, URI templates, actions, and queries. It also provides components for combing some of these, such as an action with a URI template for the URL.

This library could then be used, for example, to add actions to [HAL](http://stateless.co/hal_specification.html) documents by dropping them into the messages and pointing to the Hyperdescribe docs for [actions](https://github.com/smizell/hyperextend#action). HAL and other formats are designed to be extended upon, so this fits within many specs.

Here is a HAL response being extended with a Hyperextend action.

~~~json
{
  "first_name": "John",
  "last_name": "Doe",
  "_links": {
    "self": { "href": "/customers/4" },
    "next": { "href": "/customers/5" }
  },
  "_actions": {
    "edit": {
      "title": "Edit Customer",
      "href": "/customers/4",
      "method": "PUT",
      "bodyParams": [
        {
          "name": "first_name",
          "type": "string",
          "label": "First Name"
        },
        {
          "name": "last_name",
          "type": "string",
          "label": "Last Name"
        }
      ]
    }
  }
}
~~~

Hyperextend can also be used to extend profile formats and other documentation to allow for more formats to cover more ground.

I'm also currently working on a project called [Hyperschema.org](http://hyperschema.org/), which is a standard vocabulary for describing hypermedia formats. Because I've broken out each media type into reusable components, these can also be reused to extend other formats. I mention this because it means you can also mix and match formats, such as using [Siren actions](http://hyperschema.org/mediatypes/siren#/definitions/action) to extend HAL documents if you so desired.

Hopefully this allows hypermedia formats to be more flexible and make it easier to solve problems where some formats didn't seem like the right choice. If you have any questions or ideas, please add a Github issue or get in contact with me!
