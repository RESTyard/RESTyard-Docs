---
title: Home
layout: home
nav_order: 0
---

# RESTyard

This project consists of a set of Extensions for ASP.NET Core projects. The purpose is to
assist in building restful Web services using the [Siren Hypermedia Format](https://github.com/kevinswiber/siren) with much less coding.
Using the Extensions it is possible to return HypermediaObjects as C# classes. Routes for HypermediaObjects and Actions are built using extended attribute routing.

Of course there might be some edge cases or flaws. Comments, suggestions, remarks and criticism are very welcome.

For a first feel there is a demo project called [CarShack](https://github.com/bluehands/WebApiHypermediaExtensions/tree/master/Source/CarShack) which shows a great portion of the Extensions in use. It also shows how the routes were intended to be designed, although this is not enforced.

To develop C# client applications there is a generic REST client.

## On nuget.org

The Extensions: [https://www.nuget.org/packages/RESTyard.AspNetCore](https://www.nuget.org/packages/RESTyard.AspNetCore)

The Client: [https://www.nuget.org/packages/RESTyard.Client/](https://www.nuget.org/packages/RESTyard.Client/)

## UI client

A RESTyard API can be access using a generic UI client: [HypermediaUi](https://github.com/RESTyard/HypermediaUI)

## Predecessor

Formerly this project was named `Web Api Hypermedia Extensions`. We move to a new name so we have a whole namespace for partner projects and it is more easy to find.
