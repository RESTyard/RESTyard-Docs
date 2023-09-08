---
layout: default
title: Options
nav_order: 8
---

# Options

For some configuration you can pass a `HypermediaExtensionsOptions` object to `AddHypermediaExtensions` method.

- `ReturnDefaultRouteForUnknownHto` if set to `true` the `IHypermediaRouteResolver` will return a default route (see: `DefaultRouteSegmentForUnknownHto`) if a HTO's route is unknown.
  This is useful during development time when first writing some HTO's and not all controllers are implemented. Default is `false`.

- `DefaultRouteSegmentForUnknownHto` a `string` which will be appended to `<scheme>://<authority>/` as default route.

- `AutoDeliverJsonSchemaForActionParameterTypes`: if set to `true` (default) the routes to action parameters (implementing `IHypermediaActionParameter`) will be generated automatically except when there is a explicit rout created.

- `ImplicitHypermediaActionParameterBinders`: if set to `true` (default) actions receiving a `IHypermediaActionParameter` will have a automatic binder added. 
  The binder allows to use the `KeyFromUriAttribute` attribute. See: {% link 02-Get-started/Url-key-extraction.md %}
