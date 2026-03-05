---
layout: default
title: Get started
parent: Getting started
nav_order: 1
---

# Get started
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

## 1. Install the NuGet package

```bash
dotnet add package RESTyard.AspNetCore
```

## 2. Register RESTyard in your DI container

In `Program.cs`:

```csharp
builder.Services.AddControllers();

builder.Services.AddHypermediaExtensions(o =>
{
    o.ReturnDefaultRouteForUnknownHto = true; // useful during development
    // RESTyard scans this assemblies at startup to find all HTO types and attributed routes
    o.ControllerAndHypermediaAssemblies = [typeof(Program).Assembly];
});
```

## 3. Define your first HTO

Create a simple HTO (Hypermedia Transfer Object) implementing `IHypermediaObject`:

```csharp
[HypermediaObject(Title = "Entry to the API", Classes = new[] { "Entrypoint" })]
public class EntrypointHto : IHypermediaObject
{
    [Relations([DefaultHypermediaRelations.Self])]
    public ILink<EntrypointHto> Self { get; set; }

    [Relations(["Customers"])]
    public ILink<CustomersRootHto> Customers { get; set; }

    public EntrypointHto()
    {
        Self = Link.To(this);
        Customers = Link.ByKey<CustomersRootHto>(null);
    }
}

[HypermediaObject(Title = "The Customers API", Classes = new[] { "CustomersRoot" })]
public class CustomersRootHto : IHypermediaObject
{
    public int TotalCustomers { get; set; }

    [Relations([DefaultHypermediaRelations.Self])]
    public ILink<CustomersRootHto> Self { get; set; }

    public CustomersRootHto(int totalCustomers)
    {
        TotalCustomers = totalCustomers;
        Self = Link.To(this);
    }
}
```

## 4. Create controllers with attributed routes

```csharp
[Route("[controller]")]
[ApiController]
public class EntryPointController : ControllerBase
{
    [HttpGet(""), HypermediaObjectEndpoint<EntrypointHto>]
    public ActionResult Get()
    {
        return Ok(new EntrypointHto());
    }
}

[Route("Customers")]
[ApiController]
public class CustomersController : ControllerBase
{
    [HttpGet(""), HypermediaObjectEndpoint<CustomersRootHto>]
    public ActionResult Get()
    {
        return Ok(new CustomersRootHto(42));
    }
}
```

## 5. Run and see Siren output

```bash
dotnet run
```

Request `http://localhost:5000/EntryPoint` with `Accept: application/vnd.siren+json` (or no Accept header). You'll get:

```json
{
  "class": ["Entrypoint"],
  "title": "Entry to the API",
  "properties": {},
  "entities": [],
  "actions": [],
  "links": [
    {
      "rel": ["self"],
      "href": "http://localhost:5000/EntryPoint"
    },
    {
      "rel": ["Customers"],
      "href": "http://localhost:5000/Customers"
    }
  ]
}
```

All links are automatically resolved from the attributed routes — you never write URL strings.

## Next steps

- [HypermediaObject]({% link content/03-HypermediaObject.md %}) — define HTOs with properties, actions, and links
- [Links and Embedded Entities]({% link content/04-Entity-and-Links.md %}) — reference other HTOs
- [Endpoints]({% link content/05-Endpoints.md %}) — route attributes, actions, queries, and file uploads
- [Configuration]({% link content/08-Options.md %}) — configuration options
