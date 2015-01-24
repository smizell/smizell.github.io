---
layout: post
title: Rails Isn't Really RESTful
date: 2013-11-30 01:47 UTC
category: technology
tags: rest, rails
description: A look at how Rails does REST
---

Lately I've really been studying the [REST architecture](http://en.wikipedia.org/wiki/Representational_state_transfer) and have really had a lot of my views about the web completely changed, along with my hopes for where I'd love to see the web go. I've also wanted to start writing publicly more, so I thought I'd put down some thoughts and ideas I've had recently about REST.

I wanted to start with a post exploring some of the headaches I've had when building a truly RESTful API in Rails. The way Rails does REST is not really REST, and it's also somewhat counterproductive when trying to really do REST.

## What I Mean By REST

I thought I'd first clarify what I mean when I say REST. In the world of API development, REST usually includes the following concepts.

1. Pretty URLs
2. HTTP verbs for CRUD
3. Different data format types, such as JSON or XML, through the same URL

REST *can* include these concepts, though they don't really get to the heart of what REST really is. In reality, REST primarily deals with three big ideas.

1. **Resources**
2. **Representations** of those resources
3. **Hypermedia** to instruct the client on how to interact with those resources

REST comes from an idea that for systems to be scalable and distributed, clients and servers must be able to understand each other without any human intervention. You see this everyday on the web. Your browser—the client—can understand any website—the server—that serves up HTML over HTTP. It will give you links for navigating the site along with forms for adding, changing, or removing resources. You don't have to have a special client for each site you visit. It's all handled by that one client because the interfaces are uniform and because it uses a known hypermedia, HTML.

Of course, this small section does not do REST justice, so please explore other resources on this topic written by much smarter people. My main point with this, though, is that REST is not limited to the simple ideas of CRUD, HTTP verbs, and pretty URLs, but revolves around the concept of resources.

## The Issues with Rails

Here are a couple of bigger issues I came across that I felt were counterproductive in building a RESTful API. I'll go through each of these.

1. **REST is not about the database** - The difficulty here is that Rails is primarily built for making your database accessible through the web and not about resources
2. **Rails resources are not REST resources** - Rails resources that are mapped to controller actions add an unnecessary layer of complexity that took me a while to work through. Additionally, Rails controllers reference multiple resources which adds to that complexity.

### 1. REST is not about the database

This may be a straw man here, but Rails seems to ask the questions, "How can I make my database models available to my application view layer, and how can I get data from my view layer back to my database?" Rails is of course primarily a framework for building web applications, and so this makes sense.

Rails is setup as a Model-View-Controller (MVC) framework. In Rails' MVC, models are tied to your database tables, views represent data (not necessarily a model itself), and controllers act as the glue, making models available to views and receiving input from views to be stored in the models.

In light of all of this, when people talk of REST endpoints in Rails, they are usually referring to accessing a given database model through HTTP. Rails provides several routes for accessing this data that revolve around the idea of CRUD, which has its roots in the database world. As shown on the [Wikipedia page for CRUD](http://en.wikipedia.org/wiki/Create,_read,_update_and_delete#Database_applications), it seems to assert that using HTTP verbs like database calls is considered RESTful [^wikipedia].

In reality, REST is not about accessing data in your database or like a database, but rather it is about accessing resources and performing actions on those resources (which may or may not fit into the idea of CRUD). Read back through all that I just said above about MVC and CRUD and you won't see anything about resources, resource representations, or hypermedia, which are all at the heart of REST.

If Rails was built around the idea of resources instead of starting with the database and working to expose it, we'd be much closer to have a better web with better APIs that harness this idea of discoverability, and as you will see below, it would make things a lot simpler.

### 2. Rails resources are not REST resources

This was one of the harder ones for me to get past, and it was the idea that a controller is not equal to a resource.

Rails has a routes file for directing requests to their proper controllers. I'll start with the code for the route and work our way into the controller. I will use the example from [Rails' docs on routing](http://guides.rubyonrails.org/routing.html).

In your routes file, you have something as simple as the line below.

~~~ruby
resources :photos
~~~

This gives you several different routes that point to your PhotosController. It gives you two primary URLs for your resource(s):

* **/photos** which is a collection of photos
* **/photos/:id** which is a specific photo resource

When it comes down to it, **those are both different resources**. A collection of items should be treated as a different resource than the actual item itself. Grasping this took me a long while since I had to understand I needed two different types of representations.

In this case, posting to a collection resource allows you to add a resource to that collection. You may want to provide links in your response that filter down that collection resource or even paginate it. The important part of this is that you have to keep it straight even though your controller may look like this below.

~~~ruby
class PhotosController < ApplicationController
  def index
  end

  def new
  end

  def create
  end

  def show
  end

  def edit
  end

  def update
  end

  def destroy
  end
end
~~~

#### Lining things up

This is all in one controller, but here is how they line up with actual resources.

##### Photo collection resource at /photos

* **index** is used to GET a collection of photos
* **create** is used when sending a POST request to the collection to add a resource

##### Photo item resource at /photos/:id

* **show** is used to GET an item
* **update** is used when updating an item (via PATCH/PUT)
* **destroy** is used to DELETE an item

##### Form controls

These are used to display forms for creating/updating items. In most APIs, these forms are either included in the response or described in the link relations. That doesn't mean they can't be on a separate URL, though.

* **new** is a form control for adding a new photo item
*  **edit** is a form control for editing a photo item

#### Bringing it back together

This is all very confusing to keep straight. If you don't understand that a collection of items and items themselves are different resources, you'll end up completely missing the idea of having a collection resource, and if you're like me and think like I did, you'll just make a JSON list of item resources.

Why does Rails have to do this with its interpretation of web resources? Why is the MVC architecture there if it complicates things so much?

## Conclusion

Rails is designed to build applications, and does a great job at this. Rails is not designed, however, to build web resources out of the box that fit into the REST architecture and constraints. You can make Rails work, but you first have to understand making web resources is not about exposing your database tables.

Rails also uses MVC, which is great for creating applications, but adds several layers of complexity from routes to controllers to views that detract from the idea of resources. It also confuses the idea of delivering representations of resources with delivering different content types for controller actions, which are both related but completely different ideas.

I'll end by saying I really enjoy Rails and use it daily, and I wrote this article to express some issues I had along the way in hopes of helping others. I have some thoughts on how things could be improved, but I don't know if I could encapsilate those thoughts any better than [Webmachine](https://github.com/seancribbs/webmachine-ruby) does, which focuses primarily on resources.

[^wikipedia]: Just because something is said on Wikipedia doesn't make it correct, but I did want to show that this is a very common thought today.
