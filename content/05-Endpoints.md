---
layout: default
title: Endpoints
nav_order: 5
---

# Endpoints
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

{: .note }
If you're migrating from an older version, the legacy attributes (`HttpGetHypermediaObject`, `HttpPostHypermediaAction`, etc.) are still supported but deprecated. The included Roslyn analyzers (RY0010–RY0015) will suggest migration to the new attributes.

The included SirenFormatter will build required links to other routes. At startup all routes attributed with the following will be placed in an internal register:

- `HypermediaObjectEndpoint<THto>` — for GET routes returning HTOs
- `HypermediaActionEndpoint<THto>(nameof(...))` — for POST/PUT/PATCH/DELETE action routes
- `HypermediaActionParameterInfoEndpoint<TParam>` — for action parameter schema routes

This means that for every HTO there must be a route with matching type.

Example from the demo project CustomersRootController:

```csharp
[HttpGet(""), HypermediaObjectEndpoint<HypermediaCustomersRootHto>]
public ActionResult GetRootDocument()
{
    return Ok(customersRoot);
}
```

The same goes for Actions. The action endpoint references the HTO that owns the action and names the action property:

```csharp
[HttpPost("CreateCustomer"),
 HypermediaActionEndpoint<HypermediaCustomersRootHto>(nameof(HypermediaCustomersRootHto.CreateCustomer))]
public async Task<ActionResult> NewCustomerAction(CreateCustomerParameters createCustomerParameters)
{
    if (createCustomerParameters == null)
    {
        return this.Problem(ProblemJsonBuilder.CreateBadParameters());
    }

    var createdCustomer = await CreateCustomer(createCustomerParameters);

    // Will create a Location header with a URI to the result.
    return this.Created(Link.To(createdCustomer));
}
```

{: .highlight }
Siren specifies that to trigger an action an array of parameters should be posted to the action route. To avoid wrapping parameters in an array class there is the SingleParameterBinder for convenience.

A valid JSON for this route would look like this:

```json
[{"CreateCustomerParameters":
   {
     "Name":"Hans Schmid"
   }
}]
```

The parameter binder also allows to pass a parameter object without the wrapping array:

```json
{"CreateCustomerParameters":
    {
      "Name":"Hans Schmid"
    }
}
```

Parameters for actions may define a route which provides additional type information to the client. These routes will be added to the Siren fields object as "class".

```csharp
[HttpGet("NewAddressType"), HypermediaActionParameterInfoEndpoint<NewAddress>]
public ActionResult NewAddressType()
{
    var schema = jsonSchemaFactory.Generate(typeof(NewAddress));
    return Ok(schema);
}
```

Also see: [URL key extraction]({% link content/06-Url-key-extraction.md %})

## Actions with prefilled values

Actions can supply prefilled values so a form is already filled with server provided content. Actions with parameters have an optional parameter:

```csharp
public class CreateQueryOp : HypermediaAction<CustomerQuery>
{
    public CreateQueryOp(Func<bool> canExecute, CustomerQuery? prefilledValues = default)
        : base(canExecute, prefilledValues)
    {
    }
}
```

## Actions with acceptable media type

The `HypermediaActionEndpoint` attribute is combined with standard HTTP method attributes. To specify an acceptable media type, use the `DefaultMediaTypes` constants:

```csharp
[HttpPost("UploadImage"),
 HypermediaActionEndpoint<HypermediaCarsRootHto>(nameof(HypermediaCarsRootHto.UploadCarImage), DefaultMediaTypes.MultipartFormData)]
```

Will be rendered to Siren as `type` on the action. Default is `application/json`.

## File upload actions

Use the `FileUploadHypermediaAction` or `ExternalFileUploadHypermediaAction` to specify a file upload. Pass `FileUploadConfiguration` to send information to the client about allowed behavior.

Controller example:

```csharp
// controller
[HttpPost("UploadImage"),
 HypermediaActionEndpoint<HypermediaCarsRootHto>(nameof(HypermediaCarsRootHto.UploadCarImage), DefaultMediaTypes.MultipartFormData)]
public async Task<IActionResult> UploadCarImage(
    [HypermediaUploadParameterFromForm]
    HypermediaFileUploadActionParameter<UploadCarImageParameters> uploadParameters)
{
    var files = uploadParameters.Files; // Access uploaded files
    var additionalParameter = uploadParameters.ParameterObject; // Access generic additional parameter <UploadCarImageParameters>
    //...
}

// action definition
public class UploadCarImageOp : FileUploadHypermediaAction<UploadCarImageParameters>
{
    public UploadCarImageOp(Func<bool> canExecute, FileUploadConfiguration? fileUploadConfiguration = null)
        : base(canExecute, fileUploadConfiguration)
    {
    }
}
```

Files are uploaded using `multipart/form-data`. The additional parameter is added as a serialized json string to the key-value-dictionary of the form.

C# client example:

```csharp
// hco definition
[HypermediaClientObject("CarsRoot")]
public partial class HypermediaCarsRootHco : HypermediaClientObject
{
    [HypermediaCommand("UploadCarImage")]
    public IHypermediaClientFileUploadFunction<CarImageHco, UploadCarImageParameters>? UploadCarImage { get; set; }
}

// usage
HypermediaCarsRootHco hco;
hco.UploadCarImage.ExecuteAsync(
    new HypermediaFileUploadActionParameter<UploadCarImageParameters>(
        FileDefinitions: [
            new FileDefinition(async () => new MemoryStream(new byte[] { 1, 2, 3, 4 }), "Bytes", "Bytes.txt"),
        ],
        new UploadCarImageParameters(...)),
    Resolver);
```

Note that the name and filename are mandatory in order for the file to be recognized as a file and not as a "normal" parameter.

## Calling external APIs using Actions

If it is necessary to call an external API and expose that call as an action there is `HypermediaExternalAction<TParameter>` and `HypermediaExternalAction` to be used as base for action types.

```csharp
public class ExternalActionNoParameters : HypermediaExternalAction
{
    public ExternalActionNoParameters(Uri externalUri, string httpMethod)
        : base(() => true, externalUri, httpMethod) { }
}

public class ExternalActionWithArgument : HypermediaExternalAction<ExternalActionParameters>
{
    public ExternalActionWithArgument(
        Uri externalUri,
        string httpMethod,
        string acceptedMediaType,
        ExternalActionParameters prefilledValues)
        : base(() => true, externalUri, httpMethod, acceptedMediaType, prefilledValues) { }
}

public class ExternalActionParameters : IHypermediaActionParameter
{
    public int AInt { get; }
    public ExternalActionParameters(int aInt) { AInt = aInt; }
}

// usage in HTO:
public ExternalActionNoParameters ExternalAction { get; init; }
    = new ExternalActionNoParameters(new Uri("http://www.example.com"), HttpMethod.POST);

public ExternalActionWithArgument ExternalActionWithArgs { get; init; }
    = new ExternalActionWithArgument(
        new Uri("http://www.example2.com"),
        HttpMethod.DELETE,
        "application/json",
        new ExternalActionParameters(3));
```

## Routes with a placeholder in the route template

For access to entities a route template may contain placeholder variables like _key_ in the example below.
If an HTO is referenced, e.g. the self link or a link to another Customer, the formatter must be able to create the URI to the linked HTO.
To properly fill the placeholder variables for such routes, the framework needs to know how to extract key values from the HTO.

### Use attributes to indicate keys

Use the `Key` attribute to indicate which properties of the HTO should be used to fill the route template variables.
If there is only one variable to fill it is enough to put the attribute above the desired HTO property.

{: .highlight }
A `HypermediaObjectEndpoint<T>` route must exist for the resolution to be added.

Example:

The route template: `[HttpGet("{key:int}"), HypermediaObjectEndpoint<MyHto>]`

The attributed HTO:

```csharp
public class MyHto : IHypermediaObject
{
    [Key]
    public int Id { get; set; }
    ...
```

If the route has more than one variable, the `Key` attribute receives the name of the related route template variable.

Example:

The route template: `[HttpGet("{brand}/{key:int}"), HypermediaObjectEndpoint<HypermediaCarHto>]`

The attributed HTO:

```csharp
[HypermediaObject(Title = "A Car", Classes = new[] { "Car" })]
public class HypermediaCarHto : IHypermediaObject
{
    // Marks property as part of the objects key so it can be mapped to route parameters when creating links.
    [Key("brand")]
    public string Brand { get; set; }

    // Marks property as part of the objects key so it can be mapped to route parameters when creating links
    [Key("key")]
    public int Id { get; set; }
    ...
}
```

### Use a Key record

Alternatively, define a nested `Key` record that derives from `HypermediaObjectKeyBase<T>`:

```csharp
public class HypermediaCarHto : IHypermediaObject
{
    public int? Id { get; set; }
    public string? Brand { get; set; }

    public record Key(int? Id, string? Brand) : HypermediaObjectKeyBase<HypermediaCarHto>
    {
        protected override IEnumerable<KeyValuePair<string, object?>> EnumerateKeysForLinkGeneration()
        {
            yield return new KeyValuePair<string, object?>("id", this.Id);
            yield return new KeyValuePair<string, object?>("brand", this.Brand);
        }
    }
}
```

### Use a custom `KeyProducer`

To use a custom `KeyProducer`, implement `IKeyProducer` and add it to the attributed route:

```csharp
[HttpGet("{key:int}"), HypermediaObjectEndpoint<HypermediaCustomerHto>(typeof(CustomerRouteKeyProducer))]
public async Task<ActionResult> GetEntity(int key)
{
    ...
}
```

The formatter will call the producer if it has an instance of the referenced object (e.g. from `Link.To()`) and passes it to `IKeyProducer.CreateFromHypermediaObject()`.
Otherwise it will call `IKeyProducer.CreateFromKeyObject()` and passes the key provided by `Link.ByKey()`.
The `KeyProducer` must return an anonymous object filled with a property for each placeholder variable to be filled in the HTO's route.

See `CustomerRouteKeyProducer` in the demo project for an example.

By design the extension encourages routes to not have multiple keys in the route template. Also only routes to an HTO may have a key. Actions related to
an HTO must be available as a sub route to its corresponding object so required route template variables can be filled for the action's host HTO.

Example:

```http
http://localhost:5000/Customers/{key}
http://localhost:5000/Customers/{key}/Move
```

## Queries

Clients shall not build query strings. Instead they post a JSON object to an action and receive the URI to the desired query result in the `Location` header.

```csharp
[HttpPost("Queries"),
 HypermediaActionEndpoint<HypermediaCustomersRootHto>(nameof(HypermediaCustomersRootHto.CreateQuery))]
public ActionResult NewQueryAction(CustomerQuery query)
{
    if (query == null)
    {
        return this.Problem(ProblemJsonBuilder.CreateBadParameters());
    }

    if (!customersRoot.CreateQuery.CanExecute())
    {
        return this.CanNotExecute();
    }

    // Will create a Location header with a URI to the result.
    return this.Created(Link.ByQuery<HypermediaCustomerQueryResultHto>(query));
}
```

There must be a companion route which receives the query object and returns the query result:

```csharp
[HttpGet("Query"), HypermediaObjectEndpoint<HypermediaCustomerQueryResultHto>]
public async Task<ActionResult> Query([FromQuery] CustomerQuery query)
{
    var queryResult = await customerRepository.QueryAsync(query);
    var resultReferences = new List<HypermediaCustomerHto>();
    foreach (var customer in queryResult.Entities)
    {
        resultReferences.Add(customer.ToHto());
    }

    var queries = NavigationQuerysBuilder.Create(query, queryResult);
    var result = new HypermediaCustomerQueryResultHto(
        queryResult.TotalCountOfEnties,
        resultReferences.Count,
        resultReferences,
        queries.next.Map(IHypermediaQuery (some) => some),
        queries.previous.Map(IHypermediaQuery (some) => some),
        queries.last.Map(IHypermediaQuery (some) => some),
        queries.all.Map(IHypermediaQuery (some) => some),
        query);

    return Ok(result);
}
```
