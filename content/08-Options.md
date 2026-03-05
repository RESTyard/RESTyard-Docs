---
layout: default
title: Options
nav_order: 8
---

# Options
{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

Pass a `HypermediaExtensionsOptions` object to `AddHypermediaExtensions` to configure the framework:

```csharp
builder.Services.AddHypermediaExtensions(o =>
{
    o.ReturnDefaultRouteForUnknownHto = true;
    o.ControllerAndHypermediaAssemblies = [typeof(EntryPointController).Assembly];
});
```

## ControllerAndHypermediaAssemblies

`Assembly[]` — Assemblies to scan for controller routes and HTO types. If none provided, the entry assembly is used.

Set this explicitly when your HTOs or controllers live in a different assembly than the entry point:

```csharp
o.ControllerAndHypermediaAssemblies = [typeof(EntryPointController).Assembly];
```

## ReturnDefaultRouteForUnknownHto

`bool` (default: `false`) — If `true`, the route resolver returns a default route when an HTO type has no corresponding endpoint, instead of throwing an exception.

Useful during development when writing HTOs before all controllers are implemented.

## DefaultRouteSegmentForUnknownHto

`string` (default: `"unknown/object/route"`) — The route segment appended to `<scheme>://<authority>/` when `ReturnDefaultRouteForUnknownHto` is `true`.

## AutoDeliverJsonSchemaForActionParameterTypes

`bool` (default: `true`) — Automatically generates routes that deliver JSON schema for action parameter types (implementing `IHypermediaActionParameter`). Custom schema routes created with `HypermediaActionParameterInfoEndpoint<T>` take precedence over auto-generated ones.

## CaseSensitiveParameterMatching

`bool` (default: `false`) — Controls whether matching type names for auto-generated parameter schema routes is case-sensitive.

## ImplicitHypermediaActionParameterBinders

`bool` (default: `true`) — Automatically adds custom model binders for all action parameters implementing `IHypermediaActionParameter`. This enables the (obsolete) `KeyFromUriAttribute` for those parameter types.

If `false`, custom binders are only added for parameters explicitly attributed with `[HypermediaActionParameterFromBody]`. See: [URL key extraction]({% link content/06-Url-key-extraction.md %})

## HypermediaConverterConfiguration

Configuration for the Siren serializer:

- `WriteNullProperties` (`bool`, default: `true`) — If `true`, properties with `null` values are written to the Siren document. Set to `false` to omit them.

```csharp
o.HypermediaConverterConfiguration = new HypermediaConverterConfiguration
{
    WriteNullProperties = false
};
```

## Replacing built-in services

Three core services can be replaced by providing an alternative type. The type must implement the corresponding interface:

- `AlternateRouteRegister` (`Type?`) — Replaces the default `IRouteRegister` implementation.
- `AlternateQueryStringBuilder` (`Type?`) — Replaces the default `IQueryStringBuilder` used to serialize query objects into URL query strings.
- `AlternateJsonSchemaFactory` (`Type?`) — Replaces the default `IJsonSchemaFactory` used to generate JSON schema for action parameters.

```csharp
o.AlternateQueryStringBuilder = typeof(MyCustomQueryStringBuilder);
```
