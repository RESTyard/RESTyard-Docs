---
layout: default
title: Key concepts
nav_order: 1
---

# Key concepts

## Server side APS.NET

RESTyard allows you to build a restful web server which responds with Siren documents without building a Siren class and assigning URIs to Links and embedded Entities. For this RESTyard provides two main components: the `HypermediaObject` class and new RouteAttributes extending the Web API RouteAttributes.

HypermediaObjects returned from Controllers will be formatted as Siren. All contained referenced HypermediaObjects (e.g. Links and embedded Entities), Actions, and Parameter types (of Actions) are automatically resolved and properly inserted into the Siren document, by looking up attributed routes.
