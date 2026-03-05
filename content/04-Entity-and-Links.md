---
layout: default
title: Embedded Entities and Links
nav_order: 4
---

# Embedded Entities and Links
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

Links and embedded entities are declared as typed properties on your HTO class, decorated with `[Relations(["..."])]`.

## Links

### Option 1: If an instance of the referenced HTO is available

Use `Link.To()` to create a link from an existing instance:

```csharp
[Relations(["NiceCar"])]
public ILink<HypermediaCarHto> NiceCar { get; set; }

// In constructor
NiceCar = Link.To(carInstance);
```

### Option 2: If no instance is available (key reference)

Use `Link.ByKey<T>()` to reference an HTO by its key without instantiating it:

```csharp
[Relations(["BestCustomer"])]
public ILink<HypermediaCustomerHto> BestCustomer { get; set; }

// In constructor — reference by key only
BestCustomer = Link.ByKey<HypermediaCustomerHto>(new HypermediaCustomerHto.Key(1));
```

The framework will resolve the URL using the key and the route registered for `HypermediaCustomerHto`. The `Key` record must derive from `HypermediaObjectKeyBase<T>` and provide the route template values. For simple cases, you can use `[Key]` attributes on HTO properties instead (see [Endpoints]({% link content/05-Endpoints.md %})).

### Option 3: If a query result should be referenced

Use `Link.ByQuery<T>()` to reference a query result HTO. The query object is serialized into the URL's query string:

```csharp
[Relations(["all"])]
public ILink<HypermediaCustomerQueryResultHto> All { get; set; }

// In constructor
All = Link.ByQuery<HypermediaCustomerQueryResultHto>(allQuery);
```

### Optional links

Links can be nullable to indicate they are conditionally present. You can use `Option<T>` from FunicularSwitch to map:

```csharp
[Relations(["OkaySite"])]
public ExternalLink? OkaySite { get; set; }

// In constructor — conditionally present
OkaySite = okaySite.Map(some => Link.External(some)).GetValueOrDefault();
```

## Embedded Entities

Use `List<IEmbeddedEntity<THto>>` to embed a list of HTOs. Populate with `EmbeddedEntity.Embed<T>()`:

```csharp
[Relations(["Customers"])]
public List<IEmbeddedEntity<HypermediaCustomerHto>> Customers { get; set; }

// In constructor
Customers = customerList
    .Select(c => EmbeddedEntity.Embed<HypermediaCustomerHto>(c))
    .ToList();
```

A single embedded entity can be nullable. If `null`, it will be omitted from the Siren output:

```csharp
[Relations(["FeaturedItem"])]
public IEmbeddedEntity<HypermediaCarHto>? FeaturedItem { get; set; }

// In constructor — conditionally present
FeaturedItem = featuredCar is not null
    ? EmbeddedEntity.Embed<HypermediaCarHto>(featuredCar)
    : null;
```

## External and Internal References

For links to sources outside the server or routes that cannot be built by the framework, use `ExternalReference` or `InternalReference` with `Link.External()`.

```csharp
[Relations(["GreatSite"])]
public ExternalLink GreatSite { get; set; }

// In constructor
GreatSite = Link.External(new HypermediaObjectReference(
    new ExternalReference(new Uri("https://www.example.com/"))
        .WithAvailableMediaType("text/html")));
```

An external reference can contain a media type or a list of media types. This is useful if a client wants to switch the media type e.g. to a download or get the resource as an image.

```csharp
// External references with media types
new ExternalReference(new Uri("https://www.example.com/")).WithAvailableMediaType("image/png")
new ExternalReference(new Uri("https://www.example.com/")).WithAvailableMediaTypes(["application/xml", "image/png"])

// Internal references (by route name) for server routes that can't be built by the framework
new InternalReference("My_Route_Name")
new InternalReference("My_Route_Name", new { routevariable1 = 1 })
new InternalReference("My_Route_Name").WithAvailableMediaType("image/png")
```
