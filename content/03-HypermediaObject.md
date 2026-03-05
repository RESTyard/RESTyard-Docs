---
layout: default
title: HypermediaObject
nav_order: 3
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

    // Actions — only included in Siren output if CanExecute() returns true
    [HypermediaAction(Name = "CustomerMove", Title = "A Customer moved to a new location.")]
    public CustomerMoveOp CustomerMove { get; set; }

    [HypermediaAction(Name = "MarkAsFavorite", Title = "Marks a Customer as a favorite buyer.")]
    public MarkAsFavoriteOp MarkAsFavorite { get; set; }

    public HypermediaCustomerHto(int id, string? name, int? age, AddressTo? address, bool isFavorite,
        CustomerMoveOp customerMove, MarkAsFavoriteOp markAsFavorite)
    {
        Id = id;
        Name = name;
        Age = age;
        Address = address;
        IsFavorite = isFavorite;
        CustomerMove = customerMove;
        MarkAsFavorite = markAsFavorite;
        PurchaseHistory = Link.ByQuery<CustomerPurchaseHistoryHto>(
            new CustomerPurchaseHistoryQuery(), new CustomerPurchaseHistoryHto.Key(id));
        Self = Link.To(this);
    }

    // Action types — derive from HypermediaAction<TParameter> or HypermediaAction (parameter-less)
    public class CustomerMoveOp : HypermediaAction<NewAddress>
    {
        public CustomerMoveOp(Func<bool> canExecute) : base(canExecute) { }
    }

    public class MarkAsFavoriteOp : HypermediaAction<MarkAsFavoriteParameters>
    {
        public MarkAsFavoriteOp(Func<bool> canExecute) : base(canExecute) { }
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

// Action parameters are records implementing IHypermediaActionParameter
public record NewAddress(AddressTo Address) : IHypermediaActionParameter;
public record AddressTo(string Street, string Number, string City, string ZipCode);
public record MarkAsFavoriteParameters(Uri Customer) : IHypermediaActionParameter;
```

**In short:**

- HTOs implement `IHypermediaObject` and are decorated with `[HypermediaObject(Title = "...", Classes = [...])]`
- Public properties will be formatted to Siren properties
- Properties which hold a class are recursively serialized (their public properties become nested JSON objects). Note that HTO-specific attributes like `[FormatterIgnoreHypermediaProperty]` are only applied on top-level properties, not inside nested classes.
- By default properties which are null will not be added to the Siren document
- It is recommended to represent optional values as `Nullable<T>`
- Links to other HTOs are typed `ILink<THto>` properties decorated with `[Relations(["..."])]` — see [Embedded Entities and Links]({% link content/04-Entity-and-Links.md %})
- Embedded entities use `List<IEmbeddedEntity<THto>>` populated with `EmbeddedEntity.Embed<T>()` — see [Embedded Entities and Links]({% link content/04-Entity-and-Links.md %})
- Properties with a `HypermediaActionBase` type will be added as Actions, but only if `CanExecute()` returns true. Action types derive from `HypermediaAction<TParameter>` (with parameter) or `HypermediaAction` (parameter-less). Required parameters will be added in the "fields" section of the Siren document.
- Action parameters implement `IHypermediaActionParameter` (records are recommended)
- Properties, Actions and HTOs themselves can be attributed e.g. to give them a fixed name:
  - `FormatterIgnoreHypermediaPropertyAttribute`
  - `HypermediaActionAttribute`
  - `HypermediaObjectAttribute`
  - `HypermediaPropertyAttribute`

  **Note:** This is only done for top level properties since the object tree is not traversed.

{: .highlight }
All HTOs used in a Link or as embedded Entity and all Actions in an HTO require that there is an attributed route for their Type. Otherwise the formatter is not able to resolve the URI and will throw an Exception.
