---
title: Web API with ServiceBus and caching
date: 2022-10-06 06:00:00 +/-0000
categories: [Architecture]
tags: [webapi,netcore,servicebus]     # TAG names should always be lowercase
---

## Idea behind... ##

In this blog post I'm going to showcase a neat way of preparing data for Web API, in cases when data comes from different application with the ability to have filtering applied. In this particular case my project has Azure Webjob that produces data snapshot which has to be exposed in Web API. In addition to exposing, Web API also has to convert that snapshot from domain model to supported data format, in this case ``XML`` in specific standard.

## Solution ##

First problem is how could we pass data to the Web API in an efficient way, where Web API could always listen for any new data. Great solution for this is Azure ServiceBus, which allows us to subscribe to topics and listen for any new data. Where on the other side, WebJob or any other data provider can publish data to. Now, in order for the API to always have the latest data, ServiceBus listener has to be set as BackgroundService in Web API, so it's always running - even if no one is requesting data from API itself.
 
At this point Web API has a ServiceBus listener that always fetches the latest data from ServiceBus topic. If a data publisher, in this case WebJob, does not already serve data in supported data format, for example ``XML``, data has to be converted from a given domain model. If there is any filtering of data required it can also be done beforehand. After data is ready, it could be cached.
 
So what do we have? Web API with background service that always listens for latest data from ServiceBus. Whenever new data comes in, it gets prepared in required data format and cached. Whenever a user requests data from Web API, Web API controller calls some kind of internal service which accesses cache, and returns the latest cached data.

![Web API with ServiceBus](/assets/img/posts/webapi/web-api-servicebus.png "Web API with ServiceBus")

## Summary ##

If possible, data preparation in background before user request, is a great way to minimize response times from Web API. Utilization of publish/subscribe, in this case Azure ServiceBus, and caching is one way of doing it.
 
Thanks for reading and happy coding!


