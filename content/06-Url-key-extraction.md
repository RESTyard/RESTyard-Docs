---
layout: default
title: URL key extraction
nav_order: 6
---

# URL key extraction

When it is required to reference an other resource as action parameter it is often required to get the identifying keys from the resources URL.
Enable `ImplicitHypermediaActionParameterBinders` to use this feature.
This can be automated by using attributes in parameters.
If there is only one key in use in the URL it can be attributed like this:

```csharp
public class FavoriteCustomer : IHypermediaActionParameter
{
    [Required]
    [KeyFromUri(typeof(HypermediaCustomer), schemaProperyName: "Customer")]
    public int CustomerId { get; set; }
}
```

- The first parameter `typeof(HypermediaCustomer)` gives the expected `HypermediaObject` so the frame work knows which route layout it should use, and there to what kind of Resource the provided URL should lead.
- The second parameter `schemaProperyName: "Customer"` is to identify the property which holds the URL in the payload JSON object.

{: .highlight }
When using `AutoDeliverJsonSchemaForActionParameterTypes` the delivered schemas are adapted so a URL property with the name `Customer` is required.

The post would look like:

```json
[
  {
    "FavoriteCustomer": {
      "Customer": "http://localhost:5000/Customers/1"
    }
  }
]
```

The binder will then extract the customers value `1` from the URL and sets it to the parameter which is attributed.

## Extracting from multiple URLs

{: .d-inline-block }

new (v4.3.0)
{: .label .label-green }

URL deconstruction can also be done for lists:

```csharp
public class FavoriteCustomer : IHypermediaActionParameter
{
    [Required]
    [KeyFromUri(typeof(HypermediaCustomer), schemaProperyName: "Customers")]
    public List<int> CustomerId { get; set; }
}

The post would look like:

```json
[
  {
    "FavoriteCustomer": {
      "Customers": [
        "http://localhost:5000/Customers/1",
        "http://localhost:5000/Customers/2"
      ]
    }
  }
]
```

## More than one key

If your resource is addressed using more than one key the binder needs additional information.

Example `HypermediaActionCustomerBuysCar.Parameter`:

``` csharp
public class Parameter : IHypermediaActionParameter
{
    [KeyFromUri(typeof(HypermediaCar), schemaProperyName: "CarUri", routeTemplateParameterName: "brand")]
    public string Brand { get; set; }
    [KeyFromUri(typeof(HypermediaCar), schemaProperyName: "CarUri", routeTemplateParameterName: "key")]
    public int CarId { get; set; }
    [Required]
    public double Price { get; set; }
}
```

Not two properties have an attribute indicating that they should be filled: `Brand` and `CarId`. Both share the same type of resource and `schemaProperyName` because the source of their value is a single URL in the JSON payload.
To configure which route template variable (see your attributed route) should be used to fill the parameter property `routeTemplateParameterName` is used.
The route template: `{brand}/{key:int}`. Be careful to match the variable name and `routeTemplateParameterName`.

The corresponding post would look like:

``` csharp
[
  {
    "HypermediaActionCustomerBuysCar.Parameter": {
      "CarUri": "http://localhost:5000/Cars/VW/2",
      "Price": 133
    }
  }
]
```

### Extracting multiple keys from multiple URLs

{: .d-inline-block }

new (v4.3.0)
{: .label .label-green }

When deconstructing multiple URLs into multiple keys it is required
that either all deconstruction targets are Lists or not. Else it would not be clear what values should be assigned.

Example:

``` csharp
public class ParameterWithLists : IHypermediaActionParameter
{
    [KeyFromUri(typeof(HypermediaCar), schemaProperyName: "CarUris", routeTemplateParameterName: "brand")]
    public List<string> Brands { get; set; }
    [KeyFromUri(typeof(HypermediaCar), schemaProperyName: "CarUris", routeTemplateParameterName: "key")]
    public List<int> CarIds { get; set; }
}
```
