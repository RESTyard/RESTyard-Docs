---
layout: default
title: Release Notes
nav_order: 9
---

# Release Notes

## RESTyard v4.1.0

- New Actions available `FileUploadHypermediaAction` and `ExternalFileUploadHypermediaAction`
  - Allows to specify a action which allows file upload
  - Configuration allows for:
    - File size limit
    - File type
    - single or multiple file upload
  - Siren will have an action containing a single field with configuration values

- Actions now have a filled class which indicates the class of the action:
  - `ParameterLessAction`
  - `ParameterAction`
  - `FileUploadAction`

## RESTyard v3.0.3

- Moved projects to .net 6
- Fixed HypermediaExternalObjectReference could not be used as Link, this is provided for convenience. A link should be build using a ExternalReference instance.
- Add `HypermediaExternalAction<TParameter>`and `HypermediaExternalAction` to allow calling external endpoints as actions.
- Removed the possibility to pass a execute lambda to `HypermediaActions` since this was violating layers in architecture. The HTO should not be concerned with logic execution.
  This lead to also removing HypermediaFunctions since HTOs do not execute logic, we do not need to provide means to return values. Therefor removed.
- Add new HTTP method available for operations:
  - PUT
- Fix serializing HTO properties which are using polymorphic serialization by e.g. casting `List<MyObjectBase>` to `List<object>`. For Enumerables of type `<object>` each item is inspected now.
- Fix serializing a property of Type. It is now serialized as the full name of the type.
- Fix `[KeyFromUri]` did not work if URL contained a controller token: `[controller]`
- Add possibility to use nested properties for [KeyFromUri]

## RESTyard v2.0.0

- Rebrand all projects to common name "RESTyard".

## WebApiHypermediaExtensions v1.10.0

- Extend `HttpDeleteHypermediaAction`, `HttpPatchHypermediaAction` and `HttpPostHypermediaAction` attributes so a media type can be configured which is accepted by the action.
Rendered as `type` in siren action.
- Relax requirement for relations list from `List<string>` to `IReadOnlyCollection<string>`

## WebApiHypermediaExtensions v1.9.0

- Add `InternalReference` which allows to build a link to a route by name
- Add capability for `ExternalReference` and  `InternalReference` to specify media types. This will be rendered as `type` on a Siren link. 
  This is intended for clients, so they can switch media type e.g. to a download or other than Siren via link.  

## WebApiHypermediaExtensions v1.8.2

- Fix no route key producer was added when template was null but controller still might contain a route template with variables

## WebApiHypermediaExtensions v1.8.1

- Bugfix: controller route templates were not used in automated RouteKeyProducers so full templates were needed on all methods.
  Now the Controller template is added to the one from the method
- Rework initialization so all classes are build from DI
  - Additional options to allow replacement of `IRouteRegister` and `IQueryStringBuilder` if needed

## WebApiHypermediaExtensions v1.8.0

- Reworked initialization of the framework so a single method call on the service collection is enough
  - more general approach
  - configuration is now done using a lambda
- Removed unnecessary ProblemJsonFormatter. Instead use the configured formatter.

## WebApiHypermediaExtensions v1.7.0

- Add new HTTP methods available for operations:
  - PATCH
  - DELETE

## WebApiHypermediaExtensions v1.6.1

- Bumped supported projects:
  - from netcoreapp3.0 to netcoreapp3.1
  - from netstandard1.6 to netstandard2.0

## WebApiHypermediaExtensions v1.6.0

- Fix: Fix options were not passed down on init, so they were not active.
- Move UrlConfiguration and converter configuration to options so it can be configured and is not hidden.
  This enables more configurations when using AddHypermediaExtensions()

## WebApiHypermediaExtensions v1.5.0

- Actions now can be created with a prefilled parameter object. Its will be passed as `value` in the `action`  fields`
  Example: `CreateCustomerAction = new HypermediaFunction<CreateCustomerParameters, Task<Customer>(CanCreateCustomer, DoCreateCustomer, new CreateCustomerParameters{Name = "John Doe"});`
- Fix: nullable enums are now also serialized as string (like enums)

## WebApiHypermediaExtensions v1.4.2

- Add `netcoreapp3.0` as target to support ASP.net Core 3.x

## WebApiHypermediaExtensions v1.4.1

- Add option to generate lowercase URLs
- FIX bug where ParameterBinder was not triggered
- Type parameter route matching is now case insensitive
- Replace [controller] and [action] token in routes so deconstructing routes works using the template matcher.
- Better exception messages
- Fix null reference exception when generating routes for controller without RouteAttribute
- Responses for ProblemJson now have media type: 'application/problem+json'
- Add SourceLink to GitHub

## WebApiHypermediaExtensions v1.4.0

### Features

- Action parameter objects no longer need to be wrapped in an array
- Automatic rout key producers added, so in most cases no RouteKeyProducer is needed
- Automatic mapping of HTO parameter URLs to parameter object values by URL decomposition, also represented in generated JsonSchema
- Option to auto deliver JsonSchema for ActionParameter types by crawling assemblies for parameter objects
- Option to deliver virtual URLs to HTOs which are not jet developed to ease development
- IEnumerable and arbitrary Objects in HTO properties are now serialized to Siren
- Configuration via `IMvcCoreBuilder`
- Work on HypermediaClient
  - Use a single HttpClient instance
  - Add basic authentication

### Fixes

- Update `Microsoft.AspNetCore.Mvc` for security reasons.

## WebApiHypermediaExtensions v1.3.0

### Features

- Routes with multiple variable templates are now supported, also the dependency on route variable names is removed. The KeyProducers now handle that. See documentation for details.
- Multiple Relations are now allowed for Links
- Add configuration option to SirenBuilder so writing null properties in the JSON output can be disabled
- Allow HypermediaObjects with no self link as specified by Siren
- An Exception is thrown if a route template has parameters but no KeyProvider is specified
- Add extension methods for convenience when working with embedded entities
- Add Exception if null is passed to HypermediaObjectReference
- Add ExternalReference to work around situations where a route or URI can not be constructed by the framework.
- Added a ApiMap to the CarShack project which shows an overview on the navigation possibilities for the API
- Updated CarShack project to show new features: see 'Cars' routes with multiple variable templates
- Updated README.md

### Refactoring

- Generalize Formatter concept so there can be other Formatters
- Renamed HypermediaAction with return value to HypermediaFunction
- Simplified Siren builder
- Rename RoutKeyProducer to KeyProducer because functionality is not tied to Web API
- Remove some reflections
- Renaming for clarity
- Now using NJsonSchema to generate JSON schemas
- HypermediaQueryResult: Remove NavigationQueries from constructor
- HypermediaQueryResult: Entities are no longer added by constructor
- Rename EmbeddedEntity to RelatedEntity to make usage more general
- Cleanup solution and folder structure, so now there is only one solution
- Extracted some shared functionality to Hypermedia.Util project.

### Fixes

- Most Attributes are now sealed for performance reasons
- QueryString builder now accepts null and returns string.Empty
- Fix create customer action did not set customer name
- Fix HypermediaQueryResult exposed Query property

### Hypermedia Client Prototype

There is a new project: HypermediaClient. This is a *prototype* which explores a the possibilities of a generic client which still has strong types for Hypermedia documents. To execute it see the test project: HypermediaClient.Test. The client expects a local CarShack service to communicate with.

## WebApiHypermediaExtensions v1.2.0

- ADD: It is now possible to configure generated URIs by providing a HypermediaUrlConfig (host, scheme) for links
- ADD: QueryStringBuilder can serialize IEnumerable, so it is possible to have queries containing List<>
- ADD: QueryStringBuilder can handle DateTimeOffset
- ADD: HypermediaObjectQueryReference now accept a key
- ADD: Siren JSON creation now uses ISO 8601 format for DateTime and DateTimeOffset
- ADD: Unit test project
- UPDATE: readme.md
- CHANGE: Enum handling: If enum has no EnumMember attribute no exception is thrown anymore
- CHANGE: Serialize actions and entities of embedded entities according to Siren specification
- FIX: QueryStringBuilder did not serialize nullables
- FIX: NavigationQueryBuilder next page was build but not valid
- FIX: DisablePagination() did not set PageOffset to 0
- FIX: Siren JSON creation:
  - Primitives like integers and bools were not serialized properly (serialized as string)
  - Null properties were not serialized to null but a string "null"
  - Nested classes in properties were serialized
  - Enum values were serialized as string

## WebApiHypermediaExtensions v1.1.0

- Added relations support for embedded Entities. The entities list is now filled with EmbeddedEntity objects
- Added extension methods for easy adding of embedded Entities `AddRange(..)` and `Add(..)`
- Updated CarShack demo project
- Added net452 as target framework
- Some renaming `DefaultHypermediaLinks` -> `DefaultHypermediaRelations`
- Work on README.md

## WebApiHypermediaExtensions v1.0.1

- Added XML Comments file

## WebApiHypermediaExtensions v1.0.0 release notes

- Initial release
