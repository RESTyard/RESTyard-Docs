---
layout: default
title: HypermediaObject
nav_order: 2
---

# HypermediaObject

This is the base class for all entities (in Siren format) which shall be returned from the server. Derived types from HypermediaObjects can be thought of as kind of a DTO (Data Transfer Object). A fitting name would be HTO, Hypermedia Transfer Object. They accumulate all information which should be present in the formatted Hypermedia document and will be formatted as Siren Hypermedia by the included formatter.
An Example from the demo project CarShack:

```csharp
[HypermediaObject(Title = "A Customer", Classes = new[] { "Customer" })]
public class HypermediaCustomer : HypermediaObject
{
    private readonly Customer customer;

    // Add actions:
    // Each ActionType must be unique and a corresponding route must exist so the formatter can look it up.
    // See the CustomerController.
    [HypermediaAction(Name = "CustomerMove", Title = "A Customer moved to a new location.")]
    public HypermediaActionCustomerMoveAction MoveAction { get; private set; }

    [HypermediaAction(Title = "Marks a Customer as a favorite buyer.")]
    public HypermediaActionCustomerMarkAsFavorite MarkAsFavoriteAction { get; private set; }

    // Hides the Property so it will not be pressent in the Hypermedia.
    [FormatterIgnoreHypermediaProperty]
    public int Id { get; set; }

    // Assigns an alternative name, so this stays constant even if property is renamed
    [HypermediaProperty(Name = "FullName")]
    public string Name { get; set; }

    public int Age { get; set; }

    public string Address { get; set; }

    public bool IsFavorite { get; set; }

    public HypermediaCustomer(Customer customer)
    {
        this.customer = customer;

        Name = customer.Name;
        ...

        MoveAction = new HypermediaActionCustomerMoveAction(CanMove, DoMove);
        ...
    }
...
}

```

**In short:**

- Public Properties will be formatted to Siren Properties.
- No Properties which hold a class will be serialized
- By default Properties which are null will not be added to the Siren document.
- It is recommended to represented optional values as `Nullable<T>`
- Properties with a `HypermediaActionBase` type will be added as Actions, but only if CanExecute returns true. Any required parameters will be added in the "fields" section of the Siren document.
- Other `HypermediaObject`s can be embedded by adding them as a `HypermediaObjectReferenceBase` type to the entities collection Property (not shown in this example, see HypermediaCustomerQueryResult in the demo project).
- Links to other `HypermediaObject`s can be added to the Links collection Property, also as `HypermediaObjectReferenceBase` (not shown in this example, see HypermediaCustomersRoot in the demo project).
- Properties, Actions and `HypermediaObject`s themselves can be attributed e.g. to give them a fixed name:
  - `FormatterIgnoreHypermediaPropertyAttribute`
  - `HypermediaActionAttribute`
  - `HypermediaObjectAttribute`
  - `HypermediaPropertyAttribute`

{: .highlight }
All `HypermediaObject`'s used in a Link or as embedded Entity and all `HypermediaAction`'s in a `HypermediaObject` require that there is an attributed route for their Type. Otherwise the formatter is not able to resolve the URI and will throw an Exception.
