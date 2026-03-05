---
layout: default
title: Key concepts
parent: Getting started
nav_order: 2
---

# Key concepts

## Siren Hypermedia Format

RESTyard produces responses in the [Siren](https://github.com/kevinswiber/siren) hypermedia format. A Siren document is a JSON object containing:

- **properties** — the entity's data fields (primitives, nested objects, collections)
- **links** — navigational references to other resources (e.g. self, related collections)
- **entities** — embedded sub-entities (inline or as links)
- **actions** — available operations the client can perform (with method, href, and parameter schema)
- **class** — type hints for the client

Siren's key advantage over plain JSON is that the server tells the client *what it can do next* — available actions, navigation links, and related resources are all part of the response.

## How RESTyard works

RESTyard eliminates the need to manually build Siren JSON. Instead, you define C# classes called **HTOs** (Hypermedia Transfer Objects) and the framework handles serialization and URL resolution automatically.

The flow is:

1. You define an HTO class implementing `IHypermediaObject` with properties, typed links (`ILink<T>`), and actions (`HypermediaAction<T>`)
2. You create a controller endpoint and annotate it with `HypermediaObjectEndpoint<THto>` or `HypermediaActionEndpoint<THto>`
3. At startup, RESTyard scans all attributed routes and builds a type-to-endpoint register
4. When a controller returns an HTO via `Ok(myHto)`, the Siren formatter:
   - Serializes public properties to Siren `properties`
   - Resolves all `ILink<T>` properties to URLs using the route register → Siren `links`
   - Resolves all `IEmbeddedEntity<T>` properties → Siren `entities`
   - Resolves all `HypermediaAction` properties (where `CanExecute()` is true) to URLs → Siren `actions`

The result is a fully linked Siren document where clients can discover and navigate the API without hardcoded URLs.

To see this in action, [RESTyard-HUI](https://github.com/RESTyard/RESTyard-HUI) is a generic web UI that renders any Siren API — it reads the links, entities, and actions from the response and builds a navigable interface automatically.

## Server side components

- **`IHypermediaObject`** — marker interface for all HTOs
- **`HypermediaObjectEndpoint<THto>`** — route attribute that registers a GET endpoint for an HTO type
- **`HypermediaActionEndpoint<THto>`** — route attribute that registers a POST/PUT/PATCH/DELETE endpoint for an action
- **`ILink<THto>`** — typed link property, resolved to a URL at serialization time
- **`IEmbeddedEntity<THto>`** — typed embedded entity, serialized inline as a full Siren sub-document. Use `List<IEmbeddedEntity<THto>>` for collections, populated with `EmbeddedEntity.Embed<T>()`
- **`HypermediaAction<TParameter>`** — action property with a `CanExecute()` gate and optional parameter type
- **`IHypermediaActionParameter`** — marker interface for action parameter types (records recommended)

## Controller return patterns

Controllers use different return methods depending on the situation:

### `Ok(myHto)` — Return a Siren document

The standard return for GET endpoints. The Siren formatter serializes the HTO into a full Siren document with resolved links, entities, and actions.

```csharp
[HttpGet(""), HypermediaObjectEndpoint<HypermediaCustomersRootHto>]
public ActionResult GetRootDocument()
{
    return Ok(customersRoot);
}
```

### `this.Created(link)` — Return 201 with Location header

Used when an action creates a new resource or produces a result the client should navigate to. Returns HTTP 201 with a `Location` header pointing to the created/resulting resource. The client follows this URL to get the resource.

This is central to the hypermedia approach: **the client never builds URLs** — it receives them from the server. When a client executes an action (e.g. "create customer"), the server tells the client where the result lives via the `Location` header.

See [Endpoints]({% link content/05-Endpoints.md %}) for the different `Link` variants and examples.

### Error responses

RESTyard provides extension methods for common error cases (returning [RFC 7807](https://tools.ietf.org/html/rfc7807) Problem Details):

- **`this.Problem(problemDetails)`** — Return a `ProblemDetails` response with `application/problem+json` content type
- **`this.CanNotExecute()`** — The requested action cannot be executed (e.g. state has changed since the client received the HTO)
- **`this.UnprocessableEntity()`** — Parameters were technically valid but rejected by business logic
