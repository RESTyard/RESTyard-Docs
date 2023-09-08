---
layout: default
title: Notes on serialization
parent: Get started
nav_order: 4
---

# Notes on serialization

- Enums in `HypermediaObjects` and Action parameters can be attributed using `EnumMember` to specify fixed string names for enum values. If no attribute is present the value will be serialized using `ToString()`
- `DateTime` and `DateTimeOffset` will be serialized using ISO 8601 notation: e.g. `2000-11-22T18:05:32.9990000+02:00`

## QueryStringBuilder

Building URIs which contain a query string uses the QueryStringBuilder to serialize C# Objects (tricky). If the provided implementation does not work for you it is possible to pass an alternative to the init function.

Tested for:

- Classes, with nesting
- Primitives
- IEnumerable (List, Array, Dictionary)
- DateTime, TimeSpan, DateTimeOffset
- String
- Nullable