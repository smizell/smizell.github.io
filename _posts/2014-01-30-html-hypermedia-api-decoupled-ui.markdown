---
layout: post
title: HTML, Hypermedia API, and a Decoupled UI
date: 2014-01-30 04:04 UTC
tags:
description: A different way to approach web application design
---

This article aims to start a discussion about a different way to look at developing web applications that tries to address the many difficulties surrounding building modern Single Page Applications (SPA).  If I could sum up this idea into one thought, I'd say it is:

> A hypermedia API that has HTML as the transport media type and a decoupled UI that uses something like hijax to handle links and form submissions.

There's a lot there, so I'd like to explain this.

**Hypermedia API** is the one of the keys here. A common trend today is API-first development where you first build the API behind your application, and then build your client that consumes that API. A majority of these APIs serve JSON and use CRUD and HTTP verbs for working with resources.

The difference here with a hypermedia API is that you don't just build a regular CRUD JSON API, but you include links and potentially controls for working with resources.

There are good number of APIs that do this, and it continues to increase as media types like HAL become more popular. I would like to take this constraint a little further to say that *to get the most benefit you should use a media type that the browser natively understands, which is HTML*.

**HTML as transport media type** is one of the central themes to this idea, along with decoupling the UI. The idea here is that you use HTML as the media type for representing your resources because it is has [rich semantics](http://codeartisan.blogspot.de/2012/07/using-html-as-media-type-for-your-api.html) and it hits on quite a few of the [hypermedia factors](http://amundsen.com/hypermedia/html/).

The problem with JSON, as I'll try to show, is that the browser does not natively understand it, and by nature, JSON has no standardized way to mark up hypertext (hence why hypermedia types like HAL are needed). A JSON API cannot gracefully degrade, so a separate HTML representation type must made. Usually, this HTML representation is very limited in it's functionality and is not a full description of what a client can do with a resource.

**Decoupled UI** is another important aspect. The UI is handled several ways in web applications.

 Within the idea of [progressive enhancement](http://en.wikipedia.org/wiki/Progressive_enhancement), a very basic, semantic HTML representation is built upon to allow the application to degrade gracefully. This representation is enhanced by stylesheets and scripts, which allows the application to gracefully degrade. For more complex applications, presentation code starts to creep in to this HTML and it becomes less semantic over time. It also limits what you can do on your client as you must work within the confines of the returned HTML. Additionally, it blurs the lines between backend and frontend.

In most modern applications, HTML is usually used in several different ways in an application. First, in applications that do most of the rendering in the browser instead of on the server, the initial HTML is a means to get the application code to the client. Additionally, HTML is used for templates and views while the application code consumes a JSON API. Finally, most of these applications build a separate HTML representation in cases where there is no Javascript and for search engine crawlers.

Decoupling the UI is a different way of looking at this. Instead of augmenting and enhancing the semantic HTML, a separate UI is used for the client and the semantic HTML is hidden from the human users. The HTML returned is merely referenced by the front-end UI to get data, links, and form controls. Instead of the API returning JSON, it returns HTML for every request.

In this situation, the application gracefully degrades for those without Javascript or for crawlers, and it allows developers to then build a more advanced UI layer that uses the HTML for the data. This is because our media type we are using, HTML, is natively understood by the browser, which additionally allows us to use some of the functionality of the browser.

As a note, for a decoupled UI, HTML would be a great fit though it doesn't necessarily have to be. I would assume most web applications would have the UI layer as HTML, but it's not a necessity. HTML is primarily a way to markup a document and is not necessarily made for graphical user interfaces (thought it's still great).

**Hijax** is the way in which your application uses the browser functionality to change state through hypermedia rather than building that functionality into your application code. Jeremy Keith [coined this term way back in 2006](http://domscripting.com/blog/display/41/). This is really an optional idea for what I'm explaining here, but I felt it was interesting to include because it really turns your application into a single page application that doesn't need a router.

The thought here is that the UI layer sends click and submit events down to the hidden semantic HTML layer to change application state. These events are then handled by something like [pjax](http://pjax.heroku.com/), which would build and send the ajax requests and update the underlying semantic, decoupled HTML. Once it's complete, the data is pulled from the HTML and the UI is rendered.

In this situation, the hijax layer is an abstraction layer in the sense that it just makes requests based on the events, which allows your application and API to very HATEOAS-friendly. An example for this would be, when your UI layer needs to use a form, it takes the necessary data it wants to send, fills out the underlying and hidden form, which is then handled by the hijax layer.

Again, this is really optional, but seems to round things out nicely.

## A Brief Look at Modern Single Page Applications

I want to spend some time comparing this idea to modern SPAs to show some of it's benefits and how it solves several of the issues with most SPAs. First we'll look at the issues I see in SPAs and then we'll examine very nice SPA, [Discourse](http://www.discourse.org/).

### Some Issues with SPAs

**When your entire application is in Javascript, how do you deal with users who have Javascript disabled?** A common answer to this is "If they don't have Javascript, they can't use my app." You may have felt this pain if you browser with Javascript disabled. In a few cases, that's a fair comment given the nature of their application, but there are many SPA projects that try really hard to make their app available to those without Javascript.

To solve this, in the end, the app must include a separate HTML representation in addition to the JSON returned (given the assumption that the API primarily returns responses in a JSON media type). In some cases, this HTML rendering is limited in features and not normally a full representation of the resource that is being accessed.

**Along these same lines, how do you deal with crawlers?** The solutions above can also be employed for search engine crawlers, though some projects will go a different route by rendering the HTML on the server side with a headless browser such as [Phantom.js](http://phantomjs.org/). This along with the solutions mentioned above add an extra layer to manage.

**How do frontend designers work with your API?** Most of the time (not always!), JSON APIs are simply JSON data and are not handled with a hypermedia type that includes links to direct the client where to go next. Because of this, developers building SPAs must go to the docs to understand how to work with the resources in the API or look at the code. In other words, the API is not discoverable through a browser because your browser doesn't understand JSON.

**Is your code tied to URLs?** If you use a router that maps a URL to a controller function, your client is very tightly coupled to the URL structure of your API. If your URL structure changes, your client breaks. Additionally, if your Javascript client uses different routes than your API, you run into the issue with having multiple URLs representing a single resource, something which frameworks like Rails try to avoid.

**How many data formats do you have to maintain for your client and server?** As I've already mentioned, a normal SPA has to manage the HTML that gets the code, styles, and initial JSON to the client, along with the API serving JSON.

Some apps even maintain a separate set of HTML views for those users with Javascript disabled or for crawlers. In this case, this adds another format of data to manage. Additionally, it means that the best representations of your resources are in JSON.

**Do the server responses provide transitions for the application state, or is the logic for these transitions hardcoded into the client?** If all of this logic is in the client, it means that every client must know how to navigate around the API, and that path can never change or the clients break. It also means that the logic must be duplicated in every other client that is built.

### Case Study: Discourse

I want to take a quick look at a very nicely designed SPA called [Discourse](http://www.discourse.org/), which uses a Rails backend and an Ember.js frontend. This projects seeks to solve a few of the items listed above, but it does leave some things lacking.

When you first visit the Ember page, you get the following in your response.

* The Javascript code for the application
* The initial JSON code for the first page. On most requests to the server after the first response, you are simply requesting the JSON representation rather than all of the HTML, scripts, style sheets, and images. Since you are getting all of that in the first request, Discourse includes the initial JSON to save a request.
* Basic HTML for Javascript-disabled browsers and crawlers

Once everything is downloaded, the browser will load the Javascript, which then loads the JSON and renders the UI.

In all of this, notice what the Discourse developers must manage in this situation.

* They must manage the HTML that is sent on the initial request
* They must manage the JSON API that is used on ever subsequent request
* They must manage the basic HTML
* They must build a Javascript "browser" (or client) to consume their JSON API

And as I mentioned above in the issues, this all means that the best representation of a resource is only available to clients that understand the Discourse-flavored JSON.

## Overall Benefits of using HTML, Hypermedia, and a Decoupled UI

Here are some benefits I see in using hypermedia-first application design over common designs for SPAs.

**For the API, there is only one format to manage.** Since we are using HTML for the representation instead of JSON, clients without Javascript will be able to render the resource and use the API without referencing API docs.

Additionally, we will not need to render one format for the application, another for crawlers and noscripts, and another to get our application to the client. That is all condensed to one HTML representation that is usable by all of those users.

**This allows you to use the browser functionality instead of building a browser within a browser.** Since browsers currently cannot natively understand JSON, you have to build your own JSON browser. This adds a layer of complexity to the application that requires routing and URL building in the event that no hypermedia is returned.

**Backend developers focus on semantic, structured HTML while frontend developers build their UI focused on the presentation.** For years, we've been preached to that it's important to separate your structure from your presentation. This is difficult in many situations, because a lot of times we want to add non-semantic class names or add several layers of divs for UI purposes.

This solution separates them out perfectly, so much so that frontend developers can write non-semantic HTML in the UI layer to their heart's content. Want a table for your layout? Go for it. Want a class name called "box?" Put it in the UI layer and it will never been seen by search engines or clients using your API. This UI HTML is not being used to mark up a document, so it doesn't really matter so long as it renders correctly.

Note: I don't endorse non-semantic HTML, just using it as an example to show the separation of concerns that you achieve when separating the representational HTML from the UI HTML.

On the other hand, backend developers just build nice, clean, semantic HTML. They don't have to think about the UI elements, they just represent the resource and controls with HTML.

**If you use something like Microdata in your HTML representation, you can convert it to JSON.** [Microdata](http://diveintohtml5.info/extensibility.html) was designed with this in mind, that it could be converted to JSON or other formats. This would allow you to provide a JSON representation of your resource with only a little extra work.

**UI Negotiation** I'm not sure this benefit is limited to this proposed idea, but it is a benefit to decoupling your UI. The server could sniff out the user-agent and send a different UI for mobile, tablet, and desktop. For content negotiation, we have language negotiation and format negotiation, so why couldn't you do UI negotiation based on the user agent?

**Client functionality is not dependent on URL structure.** Since the UI is rendered based on the resource returned and not the URL, you could change your entire URL structure and your clients would work seamlessly.

I was recently working on an example of this where I built an application using this method that handled blog posts. I had a list of the posts at the URL /posts, but I wanted that same list to be at the root URL. In Rails, I pointed the root URL at the method for rendering /posts and my UI worked exactly the same as if it was at /posts. This allows you to code to resource state rather than specific URLs.

## Compared to Discourse

This is a table to show a quick comparison to some of the idea proposed in this article compares to the difficulties that Discourse tries to address. Discourse really does go the extra mile, and the issues can be traced back to the fact that their JSON is not something that browsers and crawlers can currently natively understand.

<table class="table-data">
  <thead>
    <tr>
      <td></td>
      <td>Discourse</td>
      <td>HTML, Hypermedia, Decoupled UI</td>
    </tr>
  </thead>

  <tbody>

    <tr>
      <td>Browseable API?</td>
      <td>No</td>
      <td>Yes</td>
    </tr>

    <tr>
      <td>Formats backend developers must manage</td>
      <td>JSON, HTML to get code to client, and gracefully-degrading HTML</td>
      <td>Semantic HTML</td>
    </tr>

    <tr>
      <td>Fully-functioning site when gracefully degraded?</td>
      <td>No</td>
      <td>Yes</td>
    </tr>

  </tbody>
</table>

<small>Note: Saying a site that gracefully degrades is fully-functioning may need some explaining, since you can't possibly address all of your UI components in plain, semantic HTML. The idea, though, is that you can take the same actions and follow the same links that your application can.</small>

## Conclusion

Hopefully this explains some of the benefits behind building applications and APIs with HTML as the media type and decoupling the UI from the representational HTML. Once set up, it provides a much more simplified way for building complex SPAs that rely on the hypermedia to drive the application state rather than building that functionality into the client.

## Additional Reading

* [Using HTML as the Media Type for your API](http://codeartisan.blogspot.de/2012/07/using-html-as-media-type-for-your-api.html)
* [Combining HTML Hypermedia APIs and Adaptive Web Design](http://www.jayway.com/2012/08/01/combining-html-hypermedia-apis-and-adaptive-web-design/)
* [Resource-Oriented Client Architecture](http://roca-style.org/)
* [Hijax](http://domscripting.com/blog/display/41/)
* [Designing the Microblog Media Type](http://my.safaribooksonline.com/9781449309497/id2718419?link=4710c393-f150-4818-8103-9de63121b855&cid=shareLink)
