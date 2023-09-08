---
layout: default
title: Endpoints
parent: Get started
nav_order: 3
---

# Endpoints

{:toc}

The included SirenFormatter will build required links to other routes. At startup all routes attributed with:

- `HttpGetHypermediaObject`
- `HttpPostHypermediaAction`
- `HttpDeleteHypermediaAction`
- `HttpPatchHypermediaAction`
- `HttpGetHypermediaActionParameterInfo`

will be placed in an internal register.

This means that for every `HypermediaObject` there must be a route with matching type.
Example from the demo project CustomerRootController:

``` csharp
[HttpGetHypermediaObject("", typeof(HypermediaCustomersRoot))]
public ActionResult GetRootDocument()
{
    return Ok(customersRoot);
}
```

The same goes for Actions:

```csharp
[HttpPostHypermediaAction("CreateCustomer", typeof(HypermediaFunction<CreateCustomerParameters, Task<Customer>>))]
public async Task<ActionResult> NewCustomerAction([SingleParameterBinder(typeof(CreateCustomerParameters))] CreateCustomerParameters createCustomerParameters)
{
    if (createCustomerParameters == null)
    {
        return this.Problem(ProblemJsonBuilder.CreateBadParameters());
    }

    var createdCustomer = await customersRoot.CreateCustomerAction.Execute(createCustomerParameters);

    // Will create a Location header with a URI to the result.
    return this.Created(new HypermediaCustomer(createdCustomer));
}
```

Note:
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
[HttpGetHypermediaActionParameterInfo("CreateCustomerParametersType", typeof(CreateCustomerParameters))]
public ActionResult CreateCustomerParametersType()
{
    var schema = JsonSchemaFactory.Generate(typeof(CreateCustomerParameters));
    return Ok(schema);
}
```

Also see See: [URL key extraction]({% Url-key-extraction.md %})

## Actions with prefilled values

Actions supply contain prefilled values so a form is already filled with server provided content. Actions with parameters have a optional parameter:

```csharp
public class ActionWithArgument : HypermediaAction<ActionParameter>
{
    public ActionWithArgument(Func<bool> canExecute, ActionParameter prefilledValues) : base(canExecute, prefilledValues)
    {
    }
}
```

## Actions with acceptable media type

The action attributes allow to specify a media type so it can be transmitted that a client should send the data in a acceptable format

- `HttpPostHypermediaAction`
- `HttpDeleteHypermediaAction`
- `HttpPatchHypermediaAction`
- `HttpPutHypermediaAction`

```csharp
[HttpPostHypermediaAction("my/route/template", typeof(MyOperation), AcceptedMediaType = "multipart/form-data"))]
```

Will be rendered to siren as `type` on the action. Default is `application/json`.

## File upload actions

Use the `FileUploadHypermediaAction` or `ExternalFileUploadHypermediaAction` to specify a file upload. Pass `FileUploadConfiguration` to send information to the client about allowed behavior.

Controller example:

```csharp
// controller
[HttpPostHypermediaAction("UploadImage", typeof(UploadCarImageOp), AcceptedMediaType = DefaultMediaTypes.MultipartFormData)]
public async Task<IActionResult> UploadCarImage()
{
//...
}

// action definition
public class UploadCarImageOp : FileUploadHypermediaAction
{
    public UploadCarImageOp(Func<bool> canExecute, FileUploadConfiguration fileUploadConfiguration = null) : base(canExecute, fileUploadConfiguration)
    {
    }
}
````

## Calling external APIs using Actions

If it is necessary to call a external API and expose that call as an action there is `HypermediaExternalAction<TParameter>` and `HypermediaExternalAction` to be used as base for ActionTypes.

```csharp
public class ExternalActionNoParameters :HypermediaExternalAction
{
    public ExternalActionNoParameters(Uri externalUri, HttpMethod httpMethod) 
        : base(() => true, externalUri, httpMethod) { }
}

public class ExternalActionWitParameter :HypermediaExternalAction<ExternalActionParameters>
{
    public ExternalActionWitParameter(Uri externalUri,
        HttpMethod httpMethod) 
        : base(() => true,
        externalUri,
        httpMethod,
        "myCustom/mediaType",
        new ExternalActionParameters(3)) { }
}

// usage in HTO:
public ExternalActionNoParameters ExternalActionNoParametersNoParametersTest { get; init; } = new ExternalActionNoParameters(new Uri("http://www.example1.com"), HttpMethod.POST);
public ExternalActionWitParameter ExternalActionWitParameterTestOp { get; init; }= new ExternalActionWitParameter(new Uri("http://www.example2.com"), HttpMethod.DELETE);
```

## Routes with a placeholder in the route template

For access to entities a route template may contain placeholder variables like _key_ in the example below.
If a `HypermediaObject` is referenced, e.g. the self link or a link to another Customer, the formatter must be able to create the URI to the linked `HypermediaObject`.
To properly fill the placeholder variables for such routes a `KeyProducer` is required.

### Use attributes to indicate keys

Use the `Key` attribute to indicate which properties of the HTO should be used to fill the route template variables.
If there is only one variable to fill it is enough to put the attribute above the desired HTO property.

{: .highlight }
A `HttpGetHypermediaObject` route must exists for the resolution to be added.

Example:
The route template: `[HttpGetHypermediaObject("{key:int}", typeof(MyHypermediaObject))]`
The attributed HTO:

```csharp
public class MyHypermediaObject : HypermediaObject
{
    [Key]
    public int Id { get; set; }
...
```

If the route has more than one variable, the `Key` attribute receives the name of the related route template variable.

Example:
The route template: `[HttpGetHypermediaObject("{brand}/{key:int}", typeof(HypermediaCar))]`
The attributed HTO:

```csharp
[HypermediaObject(Title = "A Car", Classes = new[] { "Car" })]
public class HypermediaCar : HypermediaObject
{
    // Marks property as part of the objects key so it is can be mapped to route parameters when creating links.
    [Key("brand")]
    public string Brand { get; set; }

    // Marks property as part of the objects key so it is can be mapped to route parameters when creating links
    [Key("key")]
    public int Id { get; set; }

    ...
}
```

### Use a custom `KeyProducer`

Us use a custom `KeyProducer` implement IKeyProducer and add it to the attributed routes:
`[HttpGetHypermediaObject("{key:int}", typeof(HypermediaCustomer), typeof(CustomerRouteKeyProducer))]` to tell the framework which `KeyProducer to use for the route.

The formatter will call the producer if he has a instance of the referenced
Object (e.g. from `HypermediaObjectReference.GetInstance()`) and passes it to the `IKeyProducer:CreateFromHypermediaObject()` function.
Otherwise it will call `IKeyProducer:CreateFromKeyObject()` and passes the object provided by `HypermediaObjectKeyReference:GetKey(IKeyProducer keyProducer)`.
The `KeyProducer` must return an anonymous object filled with a property for each placeholder variable to be filled in the `HypermediaObject`'s route, here _key_.

A `KeyProducer` is added directly to the Attributed route as a Type and will be instantiated once by the framework.
See `CustomerRouteKeyProducer` in the demo project for an example.

``` csharp
[HttpGetHypermediaObject("Customers/{key:int}", typeof(HypermediaCustomer), typeof(CustomerRouteKeyProducer))]
public async Task<ActionResult> GetEntity(int key)
{
    ...
}
```

By design the Extension encourages routes to not have multiple keys in the route template. Also only routes to a `HypermediaObject` may have a key. Actions related to
a `HypermediaObject` must be available as a sub route to its corresponding object so required route template variables can be filled for the actions host `HypermediaObject`.
Example:

```http
http://localhost:5000/Customers/{key}
http://localhost:5000/Customers/{key}/Move
```

## Queries

Clients shall not build query strings. Instead they post a JSON object to a `HypermediaAction` and receive the URI to the desired query result in the `Location` header.

``` csharp
[HttpPostHypermediaAction("CreateQuery", typeof(HypermediaAction<CustomerQuery>))]
public ActionResult NewQueryAction([SingleParameterBinder(typeof(CustomerQuery))] CustomerQuery query)
{
    ...
    // Will create a Location header with a URI to the result.
    return this.CreatedQuery(typeof(HypermediaCustomerQueryResult), query);
}
```

There must be a companion route which receives the query object and returns the query result:

``` csharp
[HttpGetHypermediaObject("Query", typeof(HypermediaCustomerQueryResult))]
public async Task<ActionResult> Query([FromQuery] CustomerQuery query)
{
    ...

    var queryResult = await customerRepository.QueryAsync(query);
    var resultReferences = new List<HypermediaObjectReferenceBase>();
    foreach (var customer in queryResult.Entities)
    {
        resultReferences.Add(new HypermediaObjectReference(new HypermediaCustomer(customer)));
    }

    var navigationQuerys = NavigationQuerysBuilder.Build(query, queryResult);
    var result = new HypermediaCustomerQueryResult(resultReferences, queryResult.TotalCountOfEnties, query, navigationQuerys);
           
    return Ok(result);
}
```
