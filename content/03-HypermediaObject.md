---
layout: default
title: HypermediaObject
parent: Building your API
nav_order: 1
---

# HypermediaObject

A HypermediaObject (HTO, Hypermedia Transfer Object) represents an entity that will be serialized as a Siren document. HTOs implement the `IHypermediaObject` marker interface and accumulate all information which should be present in the formatted Hypermedia document.

An example from the demo project CarShack:

```csharp
[HypermediaObject(Title = "A Customer", Classes = new[] { "Customer" })]
public class HypermediaCustomerHto : IHypermediaObject
{
    // Hidden from Siren output, used only for route key resolution
    [Key("id")]
    [FormatterIgnoreHypermediaProperty]
    public int Id { get; set; }

    // Assigns an alternative name, so this stays constant even if property is renamed
    [HypermediaProperty(Name = "FullName")]
    public string? Name { get; set; }
    public int? Age { get; set; }
    public AddressTo? Address { get; set; }
    public bool IsFavorite { get; set; }

    // Typed link — the framework resolves the URL automatically
    [Relations(["PurchaseHistory"])]
    public ILink<CustomerPurchaseHistoryHto> PurchaseHistory { get; set; }

    // Self link
    [Relations([DefaultHypermediaRelations.Self])]
    public ILink<HypermediaCustomerHto> Self { get; set; }

    // Actions — see the Actions page for details
    [HypermediaAction(Name = "CustomerMove", Title = "A Customer moved to a new location.")]
    public CustomerMoveOp CustomerMove { get; set; }

    public class CustomerMoveOp : HypermediaAction<NewAddress>
    {
        public CustomerMoveOp(Func<bool> canExecute) : base(canExecute) { }
    }

    // Key record for route resolution (see Endpoints page)
    public record Key(int Id) : HypermediaObjectKeyBase<HypermediaCustomerHto>
    {
        protected override IEnumerable<KeyValuePair<string, object?>> EnumerateKeysForLinkGeneration()
        {
            yield return new KeyValuePair<string, object?>("id", this.Id);
        }
    }
}

public record NewAddress(AddressTo Address) : IHypermediaActionParameter;
public record AddressTo(string Street, string Number, string City, string ZipCode);
```

**In short:**

- HTOs implement `IHypermediaObject` and are decorated with `[HypermediaObject(Title = "...", Classes = [...])]`
- Public properties will be formatted to Siren properties
- Properties which hold a class are recursively serialized (their public properties become nested JSON objects). Note that HTO-specific attributes like `[FormatterIgnoreHypermediaProperty]` are only applied on top-level properties, not inside nested classes.
- By default properties which are null will not be added to the Siren document
- It is recommended to represent optional values as `Nullable<T>`
- Links to other HTOs are typed `ILink<THto>` properties decorated with `[Relations(["..."])]` — see [Links and Embedded Entities]({% link content/04-Entity-and-Links.md %})
- Embedded entities use `List<IEmbeddedEntity<THto>>` populated with `EmbeddedEntity.Embed<T>()` — see [Links and Embedded Entities]({% link content/04-Entity-and-Links.md %})
- Actions are properties deriving from `HypermediaAction<TParameter>` or `HypermediaAction`, only included if `CanExecute()` returns true — see [Actions]({% link content/04b-Actions.md %})
- Properties, Actions and HTOs themselves can be attributed e.g. to give them a fixed name:
  - `FormatterIgnoreHypermediaPropertyAttribute`
  - `HypermediaActionAttribute`
  - `HypermediaObjectAttribute`
  - `HypermediaPropertyAttribute`

  **Note:** This is only done for top level properties since the object tree is not traversed.

{: .highlight }
All HTOs used in a Link or as embedded Entity and all Actions in an HTO require that there is an attributed route for their Type. Otherwise the formatter is not able to resolve the URI and will throw an Exception.
