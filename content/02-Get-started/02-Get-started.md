---
title: Key concepts
nav_order: 1
---

# Get started

To use the Extensions just call `AddHypermediaExtensions()` on your DI container:

``` csharp
builder.Services.AddHypermediaExtensions(o =>
{
    o.ReturnDefaultRouteForUnknownHto = true; // useful during development
});
```

To configure the generated URLs in the Hypermedia documents pass a `HypermediaUrlConfig` to `AddHypermediaExtensionsInternal()`. In this way absolute URLs can be generated which have a different scheme or another host e.g. a load balancer.
