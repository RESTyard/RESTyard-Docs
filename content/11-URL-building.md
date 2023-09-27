---
layout: default
title: URL building
nav_order: 10
---

# URL building

Internally RESTyard uses the `UrlHelper` class to create required URL. To know what endpoint a `HTO` or `Action` have we need so additional information. The RouteAttributes (`HttpGetHypermediaObject`, `HttpDeleteHypermediaAction`, etc.) contain the `HTO` `Action` class (with an optional parameter type). This is why all `HTO` and `Action` have an own type and all routes can in general only reply with one (non error) class. RESTyard needs to be able to do a mapping from `HTO` or `Action` to an endpoint.

The RESTyard Attributes extend the ASP.Net native attributes e.g. `HttpGet->HttpGetHypermediaObject` or  `HttpDelete->HttpDeleteHypermediaAction` with a required Type. This type is used to build a register which maps a `HTO` or a `HypermediaAction` to an endpoint. This allows the RESTyard serializer to find required endpoints and build the URL.

{: .highlight }
For debugging or logging routes can still have a unique name. If none is given RESTyard will generate one.

## Proxies

When behind a proxy e.g. a load balancer the current hosts IP might be not the right host to build the URLs.
ASP.Net core provides means to accept headers containing the original host. See [MDN : X-Forwarded-Host](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Host)

When the use of this header is configured the `UrlHelper` will build the corresponding URLs.

Example configuration, for more details see [Forwarded Headers Middleware options](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/proxy-load-balancer?view=aspnetcore-7.0#forwarded-headers-middleware-options)

```csharp
builder.Services.Configure<ForwardedHeadersOptions>(options =>
{

    // Map headers
    options.ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto | ForwardedHeaders.XForwardedHost;
    // Address ranges of known proxies to accept forwarded headers from.
    options.KnownNetworks.Add(new IPNetwork(IPAddress.Any, 0));
});
```
