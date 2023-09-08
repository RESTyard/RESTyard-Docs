---
layout: default
title: Embedded Entities and Links
nav_order: 3
---

# Embedded Entities and Links

References to other `HypermediaObjects` are represented by references which derive from `HypermediaObjectReferenceBase`. These references are the added to the `Links` list or the `Entities` list of a `HypermediaObject`.

{:toc}

## Option 1: If a instance of the referenced HypermediaObject is available

Use a `HypermediaObjectReference` to create a reference. This reference can then be added to the Links dictionary with an associated relation:

```cshap
Links.Add("NiceCar", new HypermediaObjectReference(new HypermediaCar("VW", 2)));
```

or the Entities list (which can contain duplicates):

```cshap
Entities.Add("NiceCar", new HypermediaObjectReference(new HypermediaCar("VW", 2)));
```

{: .highlight }
The used function is an convenience extension contained in `RESTyard.AspNetCore.Hypermedia.Extensions`

## Option 2: If no instance is available or not necessary

To allow referencing of HypermediaObjects without the need to instantiate them, for reference purpose only, there are two additional references available.

use a `HypermediaObjectKeyReference` if the object requires a key to be identified e.g. the Customers id.

```csharp
Links.Add("BestCustomer", new HypermediaObjectKeyReference(typeof(HypermediaCustomer), 1));
```

The reference requires the type of the referenced HypermediaObject, here `HypermediaCustomer` and a key which is used by the related route to identify the desired entity. The framework will pass the key object to the `KeyProducer` instance which is assigned to the HypermediaObject's route, here `CustomerRouteKeyProducer`. Explicit assignment of RouteKeyProducers is optional. `KeyAttribute` can be used alternatively on key properties of the HypermediaObject. For more details on attributed routes see [Attributed routes](## Attributed routes).

Example from the CarShack demo project `CustomerController.cs`

```csharp
[HttpGetHypermediaObject("{key:int}", typeof(HypermediaCustomer), typeof(CustomerRouteKeyProducer))]
public async Task<ActionResult> GetEntity(int key)
{
..
        var customer = await customerRepository.GetEnitityByKeyAsync(key);
        var result = new HypermediaCustomer(customer);
        return Ok(result);
...
}
```

The `CustomerRouteKeyProducer` is responsible for the translation of the domain specific key `object` to a key which is usable in the route context. It must be an anonymous object where all properties match the rout template parameters, here `{key:int}`.

```csharp
public object CreateFromKeyObject(object keyObject)
{
    return new { key = keyObject };
}
```

## Option 3: If a query result should be referenced

Use a `HypermediaObjectQueryReference` if the object requires also query object `IHypermediaQuery` to be created e.g. a result object which contains several Customers.
For a reference to a query result: `HypermediaQueryResult` it is also required to provide the query to the reference, so the link to the object can be constructed.

Example from `HypermediaCustomersRoot.cs`:

```csharp
var allQuery = new CustomerQuery();
Links.Add(DefaultHypermediaRelations.Queries.All, new HypermediaObjectQueryReference(typeof(HypermediaCustomerQueryResult), allQuery));
```

## Direct References

It might be necessary to reference a external source or a route which can not be build by the framework. In this case use the `ExternalReference` for links outside of the server or `InternalReference` for server routes. These objects work around the default route resolving process by providing its own URI or route name. It can only be used in combination with `HypermediaObjectReference`.
As additional information for clients a external reference can contain a media type or a list of media types. This is useful if a client wants to switch the media type e.g. to a download or get the resource as image.

Example references of an external site:

```csharp
Links.Add("GreatSite", new ExternalReference(new Uri("http://www.example.com/")));
Links.Add("GreatSite", new ExternalReference(new Uri("http://www.example.com/")).WithAvailableMediaType("image/png"));
Links.Add("GreatSite", new ExternalReference(new Uri("http://www.example.com/")).WithAvailableMediaTypes(new []{"application/xml", "image/png"}));

Links.Add("GreatSite", new InternalReference("My_Route_Name"));
Links.Add("GreatSite", new InternalReference("My_Route_Name", new {routevariable1 = 1}));
Links.Add("GreatSite", new InternalReference("My_Route_Name").WithAvailableMediaType("image/png"));
Links.Add("GreatSite", new InternalReference("My_Route_Name").WithAvailableMediaTypes(new []{"application/xml", "image/png"}));
```
