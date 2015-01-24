---
layout: post
title: Solving FizzBuzz with Hypermedia
date: 2014-04-26 19:52 UTC
tags: hypermedia
description: Solving the FizzBuzz problem with a hypermedia api
---

I had some free time this last week and I decided to try to solve FizzBuzz with hypermedia. I thought I'd use it to show some of the benefits of using hypermedia, along with some of the downfalls of not using it.

## First, What is FizzBuzz?
If you're not familiar with FizzBuzz, here is the idea behind it ([source](http://imranontech.com/2007/01/24/using-fizzbuzz-to-find-developers-who-grok-coding/)):

> Write a program that prints the numbers from 1 to 100. But for multiples of three print “Fizz” instead of the number and for the multiples of five print “Buzz”. For numbers which are multiples of both three and five print “FizzBuzz”.

This problem is given to a lot of interviewees for jobs to make sure they can program, and it's been [solved using lots of languages](http://rosettacode.org/wiki/FizzBuzz). I thought it'd be a fun exercise to see if I could solve it with a hypermedia API.

## The FizzBuzz Hypermedia API

You can find the API [here](http://fizzbuzzaas.herokuapp.com/), which currently supports [Siren](https://github.com/kevinswiber/siren) as the media type. The [code behind the server](https://github.com/smizell/fizzbuzz-hypermedia-server) can be found on Github.

Looking at the first response, you can see the URL to where you would go to begin the FizzBuzz, along with a couple of actions you could take. The idea behind hypermedia is that you let the server give the client directions on where it can go, rather than client having to know that itself. This is the main reason for all the hyperlinks and controls in the message.

## Hypermedia Library

To get going on this, I first needed a general library that understands a Siren hypermedia message. I needed it to know how to traverse links and submit the actions in the message. I wanted a simple Python client for Siren to do this, but couldn't find one, so [I made a basic one myself](https://github.com/smizell/fizzbuzz-hypermedia-client/blob/master/siren.py). In a perfect world, this would already exist (also, I'd definitely not recommend using my code for anything of value).

## The FizzBuzz Hypermedia Client

Since I now have something that can understand my hypermedia messages, I can start writing code to follow links through my FizzBuzz API. Here's what I wrote initially ([source](https://github.com/smizell/fizzbuzz-hypermedia-client/blob/master/fizzbuzz-hypermedia1.py)):

~~~python
from siren import SirenResource as Hyperclient

BASE_URL = "http://fizzbuzzaas.herokuapp.com"

def fizzbuzz(resource):
    """
    Prints the fizzbuzz value and follows "next" links
    """
    print resource.properties["value"]

    if resource.has_link("next"):
        fizzbuzz(resource.follow_link("next"))

def begin_fizzbuzz(resource):
    """
    Follows the first link, then hands off to fizzbuzz
    """
    if resource.has_link("first"):
        fizzbuzz(resource.follow_link("first"))

root_resource = Hyperclient(BASE_URL, path="/")
begin_fizzbuzz(root_resource)
~~~

It takes a resource, prints the fizzbuzz property `value`, sees if there is a link `next`. If there is one, it follows that link and does it all over again recursively until it doesn't find a `next` resource.

This example is really nothing special, but it shows some interesting aspects of hypermedia. First, it shows what happens when the client follows the hyperlinks given to it by the server. The server knows the next number and where to find it, and so it tells that to the client. There is really no need for the client to need to figure that out.

Next, it shows how evolvable an API can be. The client code is not tied to specific URLs, nor is it tied to any logic for following links. As the designer of the API, I'm free to change up the API without breaking the client in those respects.

## FizzBuzz Client Without Hypermedia

Consider how this would be handled if hypermedia was ignored ([source](https://github.com/smizell/fizzbuzz-hypermedia-client/blob/master/fizzbuzz-non-hypermedia1.py)):

~~~python
import requests

BASE_URL = "http://fizzbuzzaas.herokuapp.com"

def fizzbuzz(params):
    url = BASE_URL + "/fizzbuzz"
    response = requests.get(url, params=params)
    response_json = response.json()
    return response_json["properties"]["value"]

for number in range(1, 101):
    print fizzbuzz({"number": number })
~~~

Notice some of the shortcomings of not using the hypermedia. For one, my code is tied directly to URLs, so those can never change. Second, my client now has to understand the logic of where the FizzBuzz starts, where the next number is, and how to interact with a specific Siren message. There is really very little that could change in the API that would not break the client.

## Well, So What?

At this point, the FizzBuzz code that ignores the hypermedia seems a lot simpler and more straightforward. What, then, are the real benefits of using the hyperlinks?

If you looked at the first response from the API, you saw an action called "Custom FizzBuzz." The non-hypermedia client will be unable to use most of the options of this action, such as `startsAt`, `endsAt`, and `add`. This is because these fields are used by the API to tell the client what is next, but our non-hypermedia has to track these on its own.

### Non-Hypermedia Example

Let's say I wanted my non-hypermedia client to start at four, add two each time, and stop at 20. Here's some code to accomplish this ([source](https://github.com/smizell/fizzbuzz-hypermedia-client/blob/master/fizzbuzz-non-hypermedia2.py)):

~~~python
import requests

BASE_URL = "http://fizzbuzzaas.herokuapp.com"
STARTS_AT = 4
ENDS_AT = 20
ADD = 2

def fizzbuzz(params):
    url = BASE_URL + "/fizzbuzz"
    response = requests.get(url, params=params)
    response_json = response.json()
    print response_json["properties"]["value"]

    params["number"] += ADD

    if params["number"] <= ENDS_AT:
        fizzbuzz(params)

fizzbuzz({ "number": STARTS_AT })
~~~

In addition to building the URLs, as mentioned before, this approach requires this non-hypermedia client to duplicate some work that the API is already doing:

1. It has to have the logic for adding numbers to get the next number
2. It has to keep track of where to end the sequence

Not only are we duplicating logic, we are also tightly coupling our client to our API.

### Same Example in Hypermedia

For my hypermedia client, I'll add in a method to handle this new custom action, then pass some params to it ([source](https://github.com/smizell/fizzbuzz-hypermedia-client/blob/master/fizzbuzz-hypermedia2.py)):

~~~python
# All other code stays the same
def custom_fizzbuzz(root_resource, params):
    """
    Submits actions for custom fizzbuzz
    """
    resource = root_resource.take_action("custom-fizzbuzz", params)
    begin_fizzbuzz(resource)

root_resource = Hyperclient(BASE_URL, path="/")
params = { "startsAt": 4, "endsAt": 20, "add": 2 }
custom_fizzbuzz(root_resource, params)
~~~

I added about four lines of code to get the functionality to handle this custom fizzbuzz action, though none of these lines resemble the non-hypermedia client where it was adding numbers and figuring out if it was at the end.

## Embedding Resources Instead of Links

This is where the hypermedia client is really going to shine. Let's say as the API designer, I see that people are making 100 requests to solve their fizzbuzz problems, so I decide to provide a way for people to request embedded resources and reduce the number of their requests. From the example, this can be accomplished through the `embed` field on the custom fizzbuzz action.

To see where the magic happens under the hood, look at the code from the Siren library, and you'll see a `follow_link` method that looks like this ([source](https://github.com/smizell/fizzbuzz-hypermedia-client/blob/master/siren.py#L68-L82)):

~~~python
def follow_link(self, link_rel, params={}):
    """
    Follow a link to a new resource or return embedded
    """
    # If the resource is embedded, use that
    if self.has_embedded(link_rel):
        return SirenEmbedded(self.embedded(link_rel))

    # If the resource is linked, return a new resource
    for link in self.links:
        if link_rel in link["rel"]:
            return SirenResource(self.base_url, link["href"], params=params)

    # Nothing was found
    return None
~~~

This says, if the resource is embedded, use it, otherwise, follow the link. To make this work for my hypermedia client, I change the `params` to look like this ([source](https://github.com/smizell/fizzbuzz-hypermedia-client/blob/master/fizzbuzz-hypermedia3.py)):

~~~python
# All other code stays the same
root_resource = Hyperclient(BASE_URL, path="/")

params = { "embed": True }
custom_fizzbuzz(root_resource, params)
~~~

Magically, the number of requests drops from 100 to just two.

Now, I'm a little reluctant to write an example of how the non-hypermedia client would handle this because this really is a feature of the hypermedia format, Siren, that I'm using. I'll go ahead and do it anyway, though ([source](https://github.com/smizell/fizzbuzz-hypermedia-client/blob/master/fizzbuzz-non-hypermedia3.py)):

~~~python
import requests

BASE_URL = "http://fizzbuzzaas.herokuapp.com"
STARTS_AT = 4
ENDS_AT = 20
ADD = 2

def fizzbuzz(resource):
    print resource["properties"]["value"]

    if "entities" in resource:
        fizzbuzz(resource["entities"][0])

def get_embedded_fizzbuzz(params):
    params["embed"] = True
    url = BASE_URL + "/fizzbuzz"
    response = requests.get(url, params=params)
    response_json = response.json()

    fizzbuzz(response_json["entities"][0])

get_embedded_fizzbuzz({"number": STARTS_AT, "endsAt": ENDS_AT,
                       "add": ADD, "firstNumber": 4})
~~~

Even though this client benefited greatly from the hypermedia aspects of the API, it still required a total rewrite of the client, whereas the hypermedia client just worked.

## Conclusion

Hopefully this shows some of the benefits of hypermedia in your API and client. It's a simple example, but it shows how evolvable the API and client can be when using hypermedia in the API.

I also wanted to mention a few takeaways I had from this little experiment.

1. As I was building the non-hypermedia client, it felt like I was building the same functionality as my Siren hypermedia library, just in a way that coupled it with my domain logic.
2. Just about every step in the process, I had to rewrite my non-hypermedia code. It felt like ignoring the hypermedia lead to some very short-sighted code.
3. As I wrote the hypermedia client, I was more focused on my current state and the transitions to get to other states. My focus was more on how the API was designed. In my non-hypermedia client, I was just focused on getting data I needed.

In the end, it takes some work to get hypermedia right, but it seems worth it in the long run.
