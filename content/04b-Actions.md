---
layout: default
title: Actions
parent: Building your API
nav_order: 3
---

# Actions
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

Actions represent operations a client can perform on an HTO. They appear in the Siren `actions` array — but only when `CanExecute()` returns `true`.

## Defining an action

Create a nested class deriving from `HypermediaAction<TParameter>` (or `HypermediaAction` for parameter-less actions). Add it as a property on your HTO:

```csharp
[HypermediaObject(Title = "A Customer", Classes = new[] { "Customer" })]
public class HypermediaCustomerHto : IHypermediaObject
{
    [HypermediaAction(Name = "CustomerMove", Title = "A Customer moved to a new location.")]
    public CustomerMoveOp CustomerMove { get; set; }

    [HypermediaAction(Name = "MarkAsFavorite", Title = "Marks a Customer as a favorite buyer.")]
    public MarkAsFavoriteOp MarkAsFavorite { get; set; }

    // Action with parameter
    public class CustomerMoveOp : HypermediaAction<NewAddress>
    {
        public CustomerMoveOp(Func<bool> canExecute) : base(canExecute) { }
    }

    // Action with parameter
    public class MarkAsFavoriteOp : HypermediaAction<MarkAsFavoriteParameters>
    {
        public MarkAsFavoriteOp(Func<bool> canExecute) : base(canExecute) { }
    }
}
```

The `[HypermediaAction]` attribute is optional — use it to give the action a fixed name or title in the Siren output. Without it, the property name is used.

## Action parameters

Action parameters implement `IHypermediaActionParameter`. Records are recommended:

```csharp
public record NewAddress(AddressTo Address) : IHypermediaActionParameter;
public record AddressTo(string Street, string Number, string City, string ZipCode);
```

Required properties of the parameter type appear as `fields` in the Siren action. By default, RESTyard auto-generates a JSON schema endpoint for each parameter type so clients can discover the expected structure (see [Configuration]({% link content/08-Options.md %})).

## CanExecute gate

The `Func<bool>` passed to the constructor controls whether the action appears in the Siren output. This lets the server tell the client what operations are currently available:

```csharp
// In the controller or service that builds the HTO:
var customerMove = new CustomerMoveOp(canExecute: () => customer.IsActive);
```

If `CanExecute()` returns `false`, the action is omitted entirely from the response.

## Prefilled values

Actions can supply default values so a client form is pre-populated. Pass them as the second constructor argument:

```csharp
public class CreateQueryOp : HypermediaAction<CustomerQuery>
{
    public CreateQueryOp(Func<bool> canExecute, CustomerQuery? prefilledValues = default)
        : base(canExecute, prefilledValues)
    {
    }
}
```

Prefilled values are serialized into the Siren `fields` as `value`.

## Parameter-less actions

For actions without parameters, derive from `HypermediaAction` (no type parameter):

```csharp
public class MarkAsDoneOp : HypermediaAction
{
    public MarkAsDoneOp(Func<bool> canExecute) : base(canExecute) { }
}
```

## File upload actions

Use `FileUploadHypermediaAction` or `FileUploadHypermediaAction<TParameter>` for actions that accept file uploads. Pass `FileUploadConfiguration` to inform the client about allowed behavior (file size, type, single vs. multiple):

```csharp
public class UploadCarImageOp : FileUploadHypermediaAction<UploadCarImageParameters>
{
    public UploadCarImageOp(Func<bool> canExecute, FileUploadConfiguration? fileUploadConfiguration = null)
        : base(canExecute, fileUploadConfiguration)
    {
    }
}
```

See [Endpoints — File upload actions]({% link content/05-Endpoints.md %}#file-upload-actions) for the controller-side setup.

## External actions

Use `HypermediaExternalAction<TParameter>` or `HypermediaExternalAction` for actions that call an external API instead of a local endpoint:

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

// Usage in HTO:
public ExternalActionNoParameters ExternalAction { get; init; }
    = new ExternalActionNoParameters(new Uri("http://www.example.com"), HttpMethod.POST);

public ExternalActionWithArgument ExternalActionWithArgs { get; init; }
    = new ExternalActionWithArgument(
        new Uri("http://www.example2.com"),
        HttpMethod.DELETE,
        "application/json",
        new ExternalActionParameters(3));
```

External actions render with the external URI as `href` in the Siren output instead of a locally resolved route.

## Dynamic actions

If an action's parameters (or whether it has any) are determined at runtime, use `DynamicHypermediaAction`. See [Dynamic actions]({% link content/13-Dynamic-content.md %}) for details.

## Action types summary

| Base class | Use case |
|---|---|
| `HypermediaAction` | Parameter-less action |
| `HypermediaAction<TParameter>` | Action with typed parameters |
| `FileUploadHypermediaAction` | File upload without extra parameters |
| `FileUploadHypermediaAction<TParameter>` | File upload with extra parameters |
| `HypermediaExternalAction` | External API call, no parameters |
| `HypermediaExternalAction<TParameter>` | External API call with parameters |
| `DynamicHypermediaAction` | Runtime-determined parameters |
