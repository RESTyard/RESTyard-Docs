---
layout: default
title: Recommendations for route design
parent: Get started
nav_order: 4
---

# Recommendations for route design

The extensions were build with some ideas about how routes should be build in mind. The Extensions do not enforce this design but it is useful to know the basic ideas.

- The API is entered by a root document which leads to all or some of the other `HypermediaObject`'s (see `HypermediaEntryPoint` in CarShack)
Examples

```
http://localhost:5000/entrypoint
```

- Collections like `Customers` are accessed through a root object (see `HypermediaCustomersRoot` in CarShack) which handles all actions which are not related to a specific customer. This also avoids that a collection directly answers with potentially unwanted Customers.
Examples

```
http://localhost:5000/Customers
http://localhost:5000/Customers/CreateQuery
http://localhost:5000/Customers/CreateCustomer
```

- Entities are accessed through a collection but do not host child Entities. These should be handled in their own collections. The routes to the actual objects should not matter, so no need to nest them. This helps to flatten the Controller hierarchy and avoids deep routes. If a placeholder variable is required in the route template name it _key_ (see Known Issues below).
Examples

```
http://localhost:5000/Customers/1
http://localhost:5000/Customers/1/Move
```
