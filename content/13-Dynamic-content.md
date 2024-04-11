---
layout: default
title: Dynamic content
nav_order: 12
---

# Dynamic content

Although the framework is build on the intend of providing type information as much as possible it can be necessary to deviate from this approach.

## Dynamic Actions
{: .d-inline-block }

(v4.2.0)
{: .label .label-green }

In certain scenarios an actions parameters (or not having any) is determined at runtime. This special case can be implemented using `DynamicHypermediaAction`.
This action allows for a runtime dynamic schema to be retrieved. To decide what schema is desired it is required to pass runtime values to the schema routes.
`DynamicHypermediaAction` has the property `SchemaRouteKeys` which accepts an object which will be passed to route generation so route keys can be filled with values.
This also requires a custom route for the dynamic schemas using `[HttpGetHypermediaActionParameterInfo]`.
For this to work the custom type route has to:

- have route keys which match the properties in `SchemaRouteKeys` so they can be filled.
- the type route is identified by a `IHypermediaActionParameter` used as marker to find the route.

Prefilled values can be provided as object which will be serialized and as string which has to contain a valid json object.

Example:
```csharp

// Action route, retreiving raw json
[HttpPostHypermediaAction("/MyDynamicAction", typeof(MyDynamicOp))]
public ActionResult MyDynamicAction([FromBody] JsonElement rawObject)
{
    // check rawObject if it matches your dynamic schema if required
    ...
    return Ok();
}

// Dynamic schema route, will work next to default generated ones
[HttpGetHypermediaActionParameterInfo("MyGenericParametersType/{schemaKey1}/{schemaKey2}", typeof(MyGenericParameters))] 
public ActionResult MyGenericParametersType([FromRoute] string schemaKey1, [FromRoute] string schemaKey2)
{
    // dynamic keys passed to the action are passed here by build URL so the schema can be selected 
    var schema = GetMyDynamicSchema(schemaKey1, schemaKey2);
    return Content(schema, DefaultMediaTypes.ApplicationJson);
}

// Action definition for HTO
public class MyDynamicOp : DynamicHypermediaAction<GenericProcessingConfigureParameters>
{
    // accepts dynamic values relevant for schema selection
    public ConfigureProcessingForJobOp(string schemaKey1Value, string schemaKey2Value, bool hasParameters, object? prefilledValues = null) 
        : base(hasParameters, prefilledValues)
    {
        // property names have to match the route for MyGenericParameters type, will be used to build the URL
        SchemaRouteKeys = new {schemaKey1=schemaKey1Value, schemaKey2= schemaKey2Value};
    }
}

// only used as marker class so custom type route is selected
public class MyGenericParameters : IHypermediaActionParameter { }

// Usage in the HTO
...
// assign the configured action with runtime information
// create prefilled values, three options
var prefilledValues = "{...}" // JSON object as string
prefilledValues = myObject; // static object
prefilledValues = new {property=1}; // anonymous object

MyDynamicOp = new MyDynamicOp(schemaKey1Value, schemaKey2Value, currentActionHasParameters(), prefilledValues);
```
