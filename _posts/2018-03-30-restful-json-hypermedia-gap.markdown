---
layout: post
title: RESTful JSON and the Hypermedia Gap
date: "2018-03-28 22:36:09 -0600"
---

We've gotten ourselves into a pickle with the term "REST." To the majority of people in the world, REST means HTTP, nice URLs, CRUD, and JSON. I'll call this view REST Common. To a separate minority, REST means exactly how Roy Fielding defined it in his thesis. I'll call this REST Proper. The gap between the two views is wide.

This separation in definitions was recognized by the REST Proper world, and they used term "hypermedia API" to differentiate. I think this was a good move. But while they added this term for REST Proper, they never gave a term for REST Common—it wasn't necessary at the time. But after all this time, we still see people telling REST Common people they aren't really doing REST. Terms like "RESTish" have popped up to aid the corrections, but the only ones who knew what that term means are the REST Proper people.

## The Hypermedia Gap

Because of this, the two groups are unable to have a meaningful conversation about the term "REST" unless the REST Common people come to a full understanding of REST Proper. This takes time and lots of digging for people new to REST Proper (I know from experience—I hail from the REST Common world). The knowledge one needs for for this jump isn't condensed for easy consumption, and it's spread across resources like blog posts, book, RPCs, and videos.

Not only can we not talk about REST across the gap, we can't even work together on the same API. The workflow and toolset for REST Common is incompatible with that of REST Proper. The former uses `application/json` and HTTP-specific documentation while the latter uses hypermedia formats, link relations, and profiles. We don't yet have a unifying solution for these paradigms.

The problem here is the gap. The problem isn't the technology or the people, but it's the fact that there is division with no means of reconciliation.

## Coming to Terms

Personally, I decided to give up the fight for the meaning of REST while I start thinking of ways to bridge the technical gap. I've found myself using "REST" for both REST Proper and REST Common, and I reach for "RESTful" most often in discussion on API patterns. I've found "RESTful" to be a term that crosses the gap (for now).

Other terms like "link" and "navigation" have helped the discussion because of familiarity. These terms allow for discussion around the hypermedia constraint of Fielding's thesis without all the baggage the term "hypermedia" brings. I'm using later in discussions, and more as a characteristic of an API rather than its defining quality.

## An Attempted Solution

[RESTful JSON](http://restfuljson.org/) was our attempt to bridge the gap both with vocabulary and implementation. It was a pattern that already existed in the REST Common world. People were adding URLs in their JSON as "links" and getting some of the same benefits as the REST Proper world was promising—they just weren't using the same terms or technology. We decided on "RESTful" to differentiate between normal JSON and JSON with hyperlinks.

RESTful JSON also fits well within the REST Common workflow and toolset. People can keep using `application/json` and their favorite API description format like OpenAPI or API Blueprint. It makes affordance-centric API design accessible to ecosystems outside the REST Proper world.

So far, the name and specification for RESTful JSON have been a stumbling block for the REST Proper world, but we've found the REST Common world quickly gets the concepts and meaning. It's a tool to share some of the hypermedia benefits across the divide without requiring the currently hefty buy-in, and that has worked for people so far.

## The Cycle Won't End

I'd love to see the community bridge the gap here, because there are new gaps forming in front of our eyes. GraphQL is rising in popularity, and with the lack of ability to join the conversation of REST vs. GraphQL, hypermedia is facing the same challenges it has faced with the modern REST discussion.

Hopefully we can move the conversation forward while including more people in it. RESTful JSON is one idea on how we might be able to do it, but there are sure to be many other paths that may do it better.