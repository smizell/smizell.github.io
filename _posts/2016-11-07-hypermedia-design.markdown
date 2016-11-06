---
layout: post
title: "Thinking Through Hypermedia Design"
date: "2016-11-07 22:36:09 -0600"
---

In designing hypermedia APIs, I usually have to use a combination of description
formats and tools to capture the right things at the right times. The context
switching can be difficult, and I am sometimes left on my own when the tooling
isn't made for describing hypermedia APIs.

I thought it would be interesting to not only share my thought process for how I
design a hypermedia API, but do so by describing what I would like to see become
possible in API-specific tooling.

## The Design Process

I personally like to the think through the steps below when I'm designing a
hypermedia API.

1. First, I define scenarios for interacting with the API
1. Then, I capture workflows that describe steps users may take
1. Next, I define semantics around the API
1. Finally, I define basic implementation details

It's most important for me to think apart from HTTP methods, URLs, schemas, and
other implementation details, as they may cloud my thinking. Once I have a clear
picture of the API, I will then start plugging it into various protocols like
HTTP, but not until the end of this process.

## The Document Structure

Let's start out by laying out a structure in YAML for our made-up format that
lines up with the thought process I've outlined above. We'll then walk through
each step and fill in the important details along the way.

```yaml
interactions: {}
workflows: {}
definitions: {}
protocols: {}
```

As we go through this, let's design an example API that allows for managing
todos. We're going to keep it very simple for the sake of brevity.

## What Can Users Do With Our API?

The most common practice for designing APIs that you see is to first think
through either the resources in your API or the URLs in your API. However, this
means right from the start I skip thinking about my application. For me, it's
most helpful if the logic is the first thing I think about.

I also like for my resources to fall out as I shake the design and think through
the edges rather than defining them at the start. This keeps me from getting to
granular or defining more resources than I will need. It also forces me to
surface all of my application functionality in the beginning rather than trying
to figure out what is there.

Here are the various interactions I want to capture in this API.

```yaml
interactions:
  viewTodoCollection: {}
  createTodoItem: {}
  viewTodoItem: {}
  markTodoComplete: {}
```

### Viewing the Collection of Todos

We can now start thinking through different scenarios a user may encounter as
they interact with the API. We'll start by describing scenarios around viewing a
collection of todos.

```yaml
interactions:
  viewTodoCollection:
    description: |
      The `item` affordance is only available when there are
      items found in the lookup.
    scenarios:
      noTodos:
        when:
          lookups:
            getTodos: []
        then:
          affordances:
            - self
            - create-todo
          status: success
      withTodo:
        when:
          lookups:
            getTodos:
              - id: 1
                body: Get milk
                completed: false
        then:
          affordances:
            - self
            - item
            - create-todo
          status: success
```

I consider these scenarios as a snapshot of interactions with API. These
scenarios are very close
to [Gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin) that let's you
describe behavior. I'm wanting to capture a small glimpse of that behavior as
I'm thinking through interactions on a micro level.

In this example, I've described two different scenarios that I think are
important. I've defined the context of those scenarios as designated with 
`when` and their expected result with `then`. I'm basically
saying, when the context of a request looks like such, here is what I expect the
response to contain.

Affordances is an important concept in a hypermedia API. Affordances define
actions a user may take in a given state. For instance, in the example, there is
an `item` affordance when there are existing todos. However, the `item`
affordance is absent when there are no todos. The absence of an affordance can
tell just as much as the presence one.

Finally, I've defined a status for each scenario. The statuses can later map
back to protocol-specific statuses, but it's still important for me to think
through how the result of the scenario should be without implementation details.

### Creating a Todo

I've written down a success and error scenario for the interaction to create a
todo. 

```yaml
interactions:
  # ...
  createTodoItem:
    scenarios:
      badData:
        when:
          request:
            data: {}
        then:
          affordances:
            - create-todo
            - collection
          status: unprocessable
      created:
        when:
          request:
            data:
              body: Get milk
        then:
          affordances:
            - collection
            - mark-complete
          status: created
```

When there is bad data, I am saying the request cannot be processed. I'm also
providing an affordance to try again for creating a todo and for going back to
view the collection of todos.

When good data is provided, though, I'm going to allow the user to go back to
the collection of todos or mark the new todo complete. I'm also saying the
status of this scenario is acknowledging the todo was created, which we know
lines up with HTTP `201`. But again, I'm skipping implemenation details for now.

### Viewing a Todo Item

I've defined how to view a list of todos and how to create a new todo, and now I
need to define what an individual todo item looks like.

```yaml
interactions:
  # ...
  viewTodoItem:
    description: |
      Once an item has been completed, the `mark-complete`
      affordance is no longer available.
    scenarios:
      incompleteTodo:
        when:
          lookups:
            getTodo:
              id: 1
              body: Get milk
              completed: false
        then:
          affordances:
            - collection
            - mark-complete
          status: success
      completeTodo:
        when:
          lookups:
            getTodo:
              id: 1
              body: Get milk
              completed: true
        then:
          affordances:
            - collection
          status: success
```

There are two scenarios I'm concerned with here. I want to see the affordances
for an incomplete todo and a complete todo. You can see here how I'm not
allowing the user at this point to mark a completed item as incomplete once
they've completed, though I may want to do so later.

### Marking a Todo Complete

Lastly, I'm going to define what it looks like when someone actually marks the
todo item complete.

```yaml
interactions:
  # ...
  markTodoComplete:
    scenarios:
      success:
        when:
          lookup:
            getTodo:
              id: 1
              body: Get milk
              completed: false
        then:
          affordances:
            - self
            - collection
          status: success
```

## What are the Common Workflows for the API?

Though, I've defined all of the scenarios I'm interested right now, that may
change as time goes on. I may even jump around during the initial design process
and define scenarios as I think about them.

Just a note before going on, you may actually think differently than I do when
you're designing an API, and that's totally OK. You may want to start with this
workflow section and move to describing the interactions of the API later.
Either way, I do think the important thing is to think about how the API will be
used before getting into details.

In describing workflows, I want to be thinking through how my interactions start
to fit together. I've captured this in part in the interactions, but now I want
to be explicit in thinking through them. This step may even cause me to go back to the
interactions above and make changes to make the workflows feel correct, so it's
a way to validate my thinking.

```yaml
workflows:
  todoLifecycle:
    description: |
      Enter the API, create a todo item, and mark it complete.
    steps:
      - enter: noTodos
      - submit: create-todo
        data:
          body: Get milk
        interaction:
          name: createTodoItem
          scenario: created
      - submit: mark-complete
        interaction:
          name: markTodoComplete
          scenario: success
  viewItem:
    description: View a todo item
    steps:
      - enter: withTodo
      - follow: item
        interaction:
          name: viewTodoItem
          scenario: incompleteTodo
  goBack:
    description: Ensure a user can go back to the todo collection
    steps:
      - enter: withTodo
      - follow: item
        interaction:
          name: viewTodoItem
          scenario: incompleteTodo
      - follow: collection
        interaction:
          name: viewTodoCollection
          scenario: withTodo
```

I've defined three workflows that I think capture most of the functionality of
the API. If I find in this step that I'm not using affordances that I defined in
my interactions, I may actually remove those. This helps me be concise in my
design and remove any lose ends.

The first workflow captures the entire todo lifecycle. We enter when there are
no todos, create a todo, and then mark it complete. You can see how I have tied
these steps back to scenarios in my interactions section.

The second is for viewing an existing item. This is simple and obvious, but it's
helpful to write it down.

Lastly, I've defined how the `collection` affordance is used, which is a way for
the user to get back to their collection of todo items. That may have not been
clear before, so hopefully that clears it up.

## What Are My Application Semantics?

It's helpful to start defining the semantics I've used throughout my thought
process. Hopefully at this point I have the design solidified and feel confident
in my choices. 

```yaml
definitions:
  # Where a user enters the API
  entry: viewTodoCollection
  properties:
    body:
      description: |
        The body of a todo item describing the action to be taken
    completed:
      description: |
        Defines whether or not a todo item has been marked as complete
  affordances:
    self:
      description: Link to current resource
      specHref: https://tools.ietf.org/html/rfc4287
    collection:
      description: A collection of various items
      specHref: https://tools.ietf.org/html/rfc6573
    item:
      description: An item in a collection
      specHref: https://tools.ietf.org/html/rfc6573
    create-todo:
      description: Create a todo item
      safe: false
    mark-complete:
      description: Mark a todo item as complete
      safe: false
```

I've defined `properties` that are relevant to this API along with the
`affordances` I've used. This hopefully can be given to client designers to have
a simple list of semantics they may encounter when using the API.

## How Does the Design Map to Existing Protocols?

Now we can map our design to a protocol. This is the place we usually see
tooling encouraging designers to begin. It's of course not a bad thing to do so,
however, I find it can force you to think about the wrong things too early, and
may cause you to overlook functionality you need to explore.

```yaml
protocols:
  http:
    todoCollection:
      uriTemplate: /
      methods:
        get: viewTodoCollection
        post: createTodoItem
    todoItem:
      uriTemplate: /todos/{id}
      methods:
        get: viewTodoItem
    markTodoComplete:
      uriTemplate: /todos/{id}/mark-complete
      methods:
        post: markTodoComplete
```

## Wrapping It Up

Just to bring it back around, notice that we did not think about resources, MVC,
nouns, verbs, URLs, schemas, JSON, or HTTP until we were happy with the design.
We naturally walked toward the implementation details of our API.

The dream for me would be that once you had all of this designed out, you could
use the document to drive your implementation. There is a lot of information
that could be used for test and behavior driven development. If I could automate
testing all of my interactions, scenarios, and workflows, I'd be really happy.

Now, I believe you may be able to accomplish some of what I have here with a
combination
of [API Blueprint](https://apiblueprint.org/), [ALPS](http://alps.io/), and
Cucumber. I'd love to see formats evolve to walk designers through capturing
affordances, behavior, and workflows in a way that helps think through the
design and drive tests and implementation.

And when it comes down it, I feel designing a hypermedia API is more difficult
than designing one that focuses on resource and HTTP first. This is because the
most popular tooling just isn't geared toward it, and because hypermedia design
requires extra thought to surface logic that drives application state. My hope
here is that we can start considering improvements to our tooling while making
it simpler to define application logic through hypermedia.
