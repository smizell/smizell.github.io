---
layout: page
title: Projects
---

The projects that I spend a majority of my time on revolve around the area of REST and hypermedia. While I have interests in all aspects of hypermedia, I have focused on the client side, specifically around the idea of having a library that can provide a single, general interface to a hypermedia message. Most of these projects below are working toward that goal and are in early development with the hopes of being ready for production very soon.

## Projects

### [Halpert](https://github.com/smizell/halpert)

Halpert is a library for representing and interfacing with hypermedia formats. It provides a way for converting to and from these formats, along with methods for filtering and parsing through them in a generic way.

### [Verbose](http://verbose.readthedocs.org/en/latest/)

Verbose is a general-purpose, multi-use hypermedia format that can be used to build robust and highly-descriptive APIs. It is meant to provide a media type can be used in many areas, while also providing a model for designing hypermedia representer objects.

### [Hyperschema.org](http://hyperschema.org/)

Hyperschema.org provides a central vocabulary for describing hypermedia formats. The idea is that all hypermedia formats have basically the same elements, and Hyperschema.org unifies these media types under a single vocabulary. The purpose of this is to provide a basis for describing resources in a general way.

### [Hyperextend](https://github.com/smizell/hyperextend)

Hyperextend is an opinionated library of components built from the core vocabulary of Hyperschema.org. It can be used to extend media types or provide a basis for new ones.

### [Hyperdescribe](https://github.com/smizell/hyperdescribe)

Hyperdescribe is a hypermedia message description format for generically describing a hypermedia message.

## Experiments

While working with these projects, I've also spent time experimenting for the purpose of learning more about hypermedia or learning a new language.

### [FizzBuzz-as-a-Service](http://smizell.com/weblog/2014/solving-fizzbuzz-with-hypermedia)

This was an experiment to show the benefits of using hypermedia APIs. I hope to build more examples like this as they help show people the benefits of using hypermedia.

* [Client code](https://github.com/smizell/fizzbuzz-hypermedia-client) - Python code for consuming the FizzBuzz server
* [Server code](https://github.com/smizell/fizzbuzz-hypermedia-server) - JavaScript code for the server
* [FizzBuzz library](https://github.com/smizell/bizzfuzz) - JavaScript FizzBuzz library, used by the server

### [Maze+XML Client and Server](https://github.com/smizell/maze_client)

This is an experiment to show how a hypermedia representer could be used on the server and in the client. It provides server code for a maze and client code to solve the maze. This example shows how a representer can be built to allow a server to easily serve multiple media types, or provide ways for a client to know how to parse multiple media types, abstracting away the complexity of dealing with hypermedia formats.

### [mm48](https://github.com/smizell/mm48)

This is a Clojure library of functions for playing the game [2048](http://gabrielecirulli.github.io/2048/). This was really my first attempt to learn Clojure, and I thought a library like this would be a good learning experience. This is just for fun!

### [React-Pjax](https://github.com/smizell/react-pjax)

An experiment to show a server and UI using HTML as the transport media type, and ReactJS for the UI. The HTML in this example is separated into transport HTML and UI HTML, which decouples the UI from the resource representation. Pjax is used for requests instead of a client-side router. This concept is explored in a [blog post](http://smizell.com/weblog/2014/html-hypermedia-api-decoupled-ui).
