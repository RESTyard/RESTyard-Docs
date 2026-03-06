---
layout: default
title: Route design
parent: Building your API
nav_order: 5
---

# Route design
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

The framework does not enforce a specific route layout, but these conventions work well with RESTyard's link resolution and are used throughout the CarShack demo.

## Entry point

Every API should have a single entry point that links to all top-level resources. Clients bookmark this one URL and discover everything else through links.

```
GET /entrypoint
```

```csharp
[HypermediaObject(Title = "Entry to the API", Classes = new[] { "Entrypoint" })]
public class EntrypointHto : IHypermediaObject
{
    [Relations([DefaultHypermediaRelations.Self])]
    public ILink<EntrypointHto> Self { get; set; }

    [Relations(["Customers"])]
    public ILink<HypermediaCustomersRootHto> Customers { get; set; }

    [Relations(["Cars"])]
    public ILink<HypermediaCarsRootHto> Cars { get; set; }
}
```

The entry point is the only URL a client needs to know. All other URLs are discovered via links.

## Collection roots

Collections like "Customers" are accessed through a root HTO, not by returning a list directly. The root HTO handles collection-level actions (create, query) and links to individual entities.

```
GET  /Customers                → CustomersRootHto (links, actions, metadata)
POST /Customers/CreateCustomer → creates a customer, returns Location header
POST /Customers/Queries        → creates a query, returns Location header to result
GET  /Customers/Query?...      → query result with embedded entities
```

This avoids returning a potentially large list of entities on the collection URL itself. The client first sees what actions are available, then explicitly requests data through a query action.

## Entities and actions

Individual entities are sub-routes of their collection. Actions on an entity are sub-routes of the entity. This ensures route template variables from the entity route are available for its actions.

```
GET  /Customers/{key}       → single customer HTO
POST /Customers/{key}/Move  → action on that customer
```

{: .highlight }
Actions must be sub-routes of their owning HTO so the framework can fill route template variables. A `Move` action on customer 42 needs the `{key}` variable from `/Customers/{key}`.

## Flat hierarchy

Do not nest entities under other entities. If a customer has orders, put orders in their own collection rather than under `/Customers/{key}/Orders/{orderId}`. This keeps controllers simple and routes shallow.

```
GET /Customers/{key}   → customer HTO (with a link to their orders)
GET /Orders/{key}      → order HTO
```

The relationship between customer and orders is expressed through links in the Siren document, not through URL nesting.

## Queries and pagination

Clients should never build query strings manually. Instead, they post a query object to an action endpoint and follow the `Location` header to the result.

```
POST /Customers/Queries          → client posts query parameters as JSON
  ← 201 Location: /Customers/Query?filter=...&page=0&pageSize=10
GET  /Customers/Query?filter=... → paginated result
```

The query result HTO includes navigation links for pagination:

```csharp
[Relations(["next"])]
public ILink<HypermediaCustomerQueryResultHto>? Next { get; set; }

[Relations(["previous"])]
public ILink<HypermediaCustomerQueryResultHto>? Previous { get; set; }

[Relations(["last"])]
public ILink<HypermediaCustomerQueryResultHto>? Last { get; set; }

[Relations(["all"])]
public ILink<HypermediaCustomerQueryResultHto>? All { get; set; }
```

Nullable links naturally disappear from the Siren output — `Next` is null on the last page, `Previous` is null on the first page.

## Route template variables

Use `{key}` as the conventional name for single-key routes. For multi-key routes, use descriptive names matching the HTO's `[Key]` attributes:

```csharp
// Single key
[HttpGet("{key:int}"), HypermediaObjectEndpoint<HypermediaCustomerHto>]

// Multiple keys
[HttpGet("{brand}/{key:int}"), HypermediaObjectEndpoint<HypermediaCarHto>]
```

See [Endpoints — Routes with a placeholder]({% link content/05-Endpoints.md %}#routes-with-a-placeholder-in-the-route-template) for how to map HTO properties to route variables.

## Summary

| Pattern | Route | Purpose |
|---|---|---|
| Entry point | `GET /entrypoint` | Single API entry, links to everything |
| Collection root | `GET /Customers` | Metadata, collection-level actions |
| Create action | `POST /Customers/CreateCustomer` | Returns `Location` to created entity |
| Query action | `POST /Customers/Queries` | Returns `Location` to query result |
| Query result | `GET /Customers/Query?...` | Paginated results with nav links |
| Entity | `GET /Customers/{key}` | Single entity with actions |
| Entity action | `POST /Customers/{key}/Move` | Action on a specific entity |
