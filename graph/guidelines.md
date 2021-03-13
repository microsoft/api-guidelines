---
title: Design your API
owner: mastaffo
---

# Design your API

<!--
> <small>"It is easy to design for users who are like you, and very difficult to design for somebody unlike you. There are too many APIs that are designed by domain experts and, frankly, they are only good for domain experts. The problem is that most developers are not, will never be, and do not need to be experts in all technologies used in modern applications." -- Krzysztof Cwalina, [Framework Design Guidelines](https://www.amazon.com/Framework-Design-Guidelines-Conventions-Libraries/dp/0321545613)</small>

> <small>"Design is really an act of communication, which means having a deep understanding of the person with whom the designer is communicating." -- Donald A. Norman, [The Design of Everyday Things](https://www.amazon.com/Design-Everyday-Things-Donald-Norman/dp/1452654123)</small>

> <small>"Good design is actually a lot harder to notice than poor design, in part because good designs fit our needs so well that the design is invisible, serving us without drawing attention to itself. Bad design, on the other hand, screams out its inadequacies, making itself very noticeable." -- Donald A. Norman, [The Design of Everyday Things](https://www.amazon.com/Design-Everyday-Things-Donald-Norman/dp/1452654123)</small>
-->

> "Make the API to your library as boring as possible. You want the functionality to be interesting, not the API." -- Chris Sells, [Framework Design Guidelines][fdg]

API design is a crucial but often overlooked aspect of the API development process.

## Your most important investment

The design of your API is arguably the most important investment you will make in it. The design of your API is what creates the first impression for developers.

We want to provide our developers an incredible promise: no matter where they are in Microsoft Graph, the API style should feel familiar. Naming, casing, filtering, pagination, and more are handled the same way through all Graph APIs.

Consistency is not cheap. This principle is so important that it is listed as the second quality of a well-designed framework in [Framework Design Guidelines](https://www.safaribooksonline.com/library/view/framework-design-guidelines/9780321545671/chapter01.html#ch1):

> Good framework design does not happen magically. It is hard work that consumes lots of time and resources. If you are not willing to invest real money in the design, you should not expect to create a well-designed framework.

## Design framework

Microsoft Graph has a well-defined framework for API design. On the positive side, this well-defined framework reduces [bikeshedding](https://en.wikipedia.org/wiki/Law_of_triviality) and results in a better experience for developers who consume Graph. On the negative side, it's harder for you. You have to learn about how we do API design, and you have less "freedom" in how you design your API.

Even so, this work results in a clean and simple-to-use experience for our users. We believe 100% that this is the right thing to do for our customers, and we hope you do too.

| Section                                             | Description                                                                                                                                                                                                                                                                                          |
| --------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Basic guidance](/add/design/basic-guidance/)       | Beginners should start with [basic guidance](/add/design/basic-guidance/), where we provide an overview of the design guidelines that inform our API design. We also provide a recommended approach for people who are new to API design.                                                            |
| [Key principles](/add/design/key-principles/)       | [Key principles](/add/design/key-principles/) are our non-negotiables. These are the naming, authentication, throttling, and error guidelines that every API must adhere to.                                                                                                                         |
| [Modeling patterns](/add/design/modeling-patterns/) | We introduce, compare, and contrast different ways to model the schema of your API in [modeling patterns](/add/design/modeling-patterns/). This includes some special things we've introduced, such as [evolvable enumerations](/add/design/modeling-patterns/evolvable-enums.md), so don't miss it! |
| [Common patterns](/add/design/common-patterns/)     | The most encouraged API patterns are documented in [common patterns](/add/design/common-patterns/). Common patterns include things we would like to see every API support, such as Webhooks and deltas.                                                                                              |
| [Advanced patterns](/add/design/advanced-patterns/) | Less common patterns are listed in [advanced patterns](/add/design/advanced-patterns/).                                                                                                                                                                                                              |
| [Design rules](/add/design/gmm/)                    | The detailed list of rules enforced by our tooling and API reviewers is available in [GMM](/add/design/gmm/). This is primarily reference content and not something we expect you to read through as you are ramping up.                                                                             |

## Section Summary

Overall, the design section exists to introduce and educate you on the best practices for building and adding your new API to Microsoft Graph. By the end of this section, you should have a working CDSL file, should understand the REST, Odata, and GMM guidelines well enough to work with the GMM testing tool, and be ready to proceed to the review and build stages with a well-designed and though-out architecture.

## Additional resources

The links below provide a set of rules to think through as you design your API.

- [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines/)
- [OData Guidelines](http://www.odata.org/documentation/)
- [Microsoft Graph Documentation](https://developer.microsoft.com/en-us/graph/docs/concepts/overview)
- [Microsoft Graph Explorer](https://aka.ms/ge)

::: tip [Need help?]
If you have additional design modeling questions (Identity team only), please ask them on the [Identity API Review Team "General" channel](https://teams.microsoft.com/l/channel/19%3a1013992de7d84c68bce90f7ae69f306a%40thread.skype/General?groupId=11b6f8e9-39e6-41f0-9fdb-dbeda26d4378&tenantId=72f988bf-86f1-41af-91ab-2d7cd011db47)
:::

[fdg]: https://www.safaribooksonline.com/library/view/framework-design-guidelines/9780321545671/chapter01.html


## Basic Guidance

Web APIs are already more than 20 years old. The SOAP specification was created in 1998, and the dissertation that became REST was written in 2000. It would take many hours to truly examine the history of APIs, so let's focus on the basics.

### Resource-oriented architecture

The first thing we need to discuss is architecture. Leonard Richardson, the author of RESTful Web Services and the more recent RESTful Web APIs, classifies APIs into three architectural categories: resource oriented, RPC oriented, and hybrid (a combination of the two).

It is critical for us to understand the difference in architectures before we can meaningfully talk about RESTful APIs. The biggest difference is arguably the number of "locations" an operation can be invoked. In RPC architectures, there are comparatively few places to invoke an operation. For instance, SOAP APIs typically expose a single URL, and callers of that API send both an action (the operation to invoke) as well as a request message that contains all of the necessary parameters for invoking the operation.

REST architectures, on the other hand, have many more "locations" an operation can be invoked. REST focuses on resources, each of which have a unique location. REST uses a URI to represent this location. CRUD operations are typically attached to the resource itself or a parent of the resource. There is often no longer a need for a request message, or if there is, it typically has fewer parameters because the operation is attached to the resource itself.

We should note right up front that neither architectural style is "evil". They both have pros and cons, and both architectures can meet most needs. Consider the analogy of object-oriented programming versus functional programming. They are both valid styles, and have pros and cons, but neither is "evil".

### History of REST

The dissertation that became REST was actually called "Architectural Styles and the Design of Network-based Software Architectures". This dissertation was written by Roy Fielding in 2000, and was based in large part on his experience designing HTTP and URIs.

If we reflect for a moment on how the Web works, we will better understand RESTful APIs. The Web is, at its most basic essence, a collection of resources – an enormous collection of resources. Web browsers are able to work with all Web sites regardless of whether they were written in pure HTML, Java, Python, or .NET. This all works because of HTTP and URIs. URIs give us a means to uniquely address a resource – a Web page, an image, a stylesheet, or a form submission resource.

#### Identifying resources

The unique identifier provided as a URI allows a Web browser to understand and request a particular resource. It also allows resources to be referenced from other resources. This basic concept is known as hypermedia – the linking of resources together using URIs. The URI is useful to many other tools on the Internet, however. The URI is what allows search engines to identify and index resources. In turn, this allows us to search for resources and find the unique identifier that represents that resource. Similarly, proxies are able to cache resources by their identifier either for security, performance, or other purposes. URIs are critical to understanding a resource.

As a side note, any URI technically constitutes a resource identifier. The URI contains everything from the scheme (HTTP or HTTPS) to the fully qualified domain name to the path to the query string parameters. Furthermore, resources in the purest sense can be a collection of other resources or an individual resource. That said, for Microsoft Graph we have further constrained the definition of "resource" in two ways:

1. Resources should be individual things, such as a single person or a single task. It is useful to distinguish a collection of individual things as a "resource collection".
2. Query string parameters should not be part of the resource identifier. Most API designs include as much of the resource identification part of the URI as possible in the main URI itself.

Again, these are Graph preferences and not part of the URI spec. The URI spec uses the broader technical definition.

#### Using resources

URIs give us the ability to identify a resource. HTTP gives us the ability to do something with that resource. HTTP has a bunch of built-in methods including `GET` (typically for retrieving a resource), `POST` (often used for creating a resource), and `PATCH` (often used for updating a resource. HTTP has semantics around virtually every aspect of transport from authorization to status to how to construct the request or response headers and body. The HTTP specification is quite large and reasonably mature, so in most cases the guidance in the spec is very clear.

### Be _openionated_

Openionated = opinionated sometimes, open-minded other times

Given the overall space (public standards, specifications, technology that is used by billions of devices), it is not hard to find people who treat the space as a religion, and publicly shame others who violate (whether from ignorance or on purpose) the specification. REST APIs also have this issue, even though there is only the dissertation and not a public standard for REST.

> As Microsoft employees, we need to be respectful of others and understand that there are a variety of reasons to do something differently. So first and foremost, engage in dialogue with others and always try to view things optimistically.

#### Be opinionated when the spec is clear

That said, there are places to be more opinionated. For instance, parts of the HTTP specification are quite clear and the implications of violating the specification are severe. Let's consider two examples.

1. The HTTP specification says that `GET` should be both _safe_ and _idempotent_. _Safe_ in this case means that the request does not cause a change in server state, _idempotent_ means that the operation is repeatable many times with the same result. However, some people violate these guidelines and create new resources by sending a `GET` to `{somecollection}/new`. The problem with this design is the billions of devices that understand HTTP. In most cases, those devices understand that the result of a `GET` is cacheable because it is safe and idempotent. Unfortunately, this API design breaks those assumptions and so devices that believe they are working with proper HTTP may in fact be causing significant problems by accessing the API.
2. Similarly, the status codes associated with an HTTP response are not a good forum for creativity. The status codes in the HTTP specification (apart from `418`) all exist for very specific reasons and have benefit to the ecosystem around them. Specification authors do not make their specifications longer than necessary because it's fun to write in spec language. The error codes in the specification are what allow us to write generic clients that understand how to handle a challenge response, or `retry-after`. It is similarly critical for devices to be able to know that if they don't understand error code `474`, they can safely treat it as `400` and know that the client is sending an improper request.

#### Be open-minded when the spec is unclear

In other cases the spec is either vague, has evolved, or specifically identifies alternative ways of achieving something. For instance, when the HTTP spec was first created, there was no `PATCH` method. Architects and developers found the need for this method over time, especially with the advent of APIs. The semantic of `PATCH` is that it allows a partial replacement to update a resource. The alternative, `PUT`, requires the full resource to be sent, and the server replaces the resource entirely. In some cases this is inefficient, in other cases it actually doesn't work (for example, if the resource has server-computed values). In any case, the addition of `PATCH` as a different specification gives API developers a choice of how to support update. Guidance is especially necessary when there are multiple conflicting or confusing means of achieving something. (For what it's worth, we recommend always supporting `PATCH` and additionally supporting `PUT` if it makes sense in your scenario.)


## Key Principles

Consistent naming is foundational for API usability. JSON is the leading serialization format for HTTP APIs, so this guidance is designed with JavaScript in mind.

Property and type names appear in URLs and payloads.

::: tip REFERENCE
These guidelines  draw on the [naming section](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#17-naming-guidelines) of the [Microsoft REST API Design Guidelines](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md). See also "Naming" in [GMM Level 1](https://msgo.azurewebsites.net/add/design/gmm/gmm-level-1.html).

The .NET [Framework Design Guidelines](https://www.safaribooksonline.com/library/view/framework-design-guidelines/9780321545671/chapter03.html#ch3) provide rationale for naming guidelines.

:::

### General Guidelines

::: tip ✔ DO use `lowerCamelCase` for _all_ names.

- Right: `automaticRepliesStatus`.
- Wrong: `kebab-case` or `snake_case`.

:::

::: warning ✖ AVOID redundant words in names.

- Right: `/places/{id}/`**_type_** and `/phones/{id}/`**_number_**
- Wrong: `/places/{id}/`_**placeType**_ and `/phones/{id}/`**_phoneNumber_**

:::

::: warning ✖ AVOID using brand names in type or property names.

- Right: `chat`
- Wrong: `teamsChat`

:::

::: warning ✖ AVOID using acronyms or abbreviations unless they are broadly understood.

- Right: `url` or `htmlSignature`
- Wrong: `msodsUrl` or `dlp`

:::

::: tip ✔ DO use singular nouns for type names.

- Right: `address`
- Wrong: `addresses`

:::

::: tip ✔ DO use plural nouns for collections (for listing a type or collection properties).

- Right: `addresses`
- Wrong: `address`

:::

::: tip ✔ DO pluralize the noun even when followed by an adjective (a "postpositive").

- Right: `passersby` or `mothersInLaw`
- Wrong: `notaryPublics` or `motherInLaws`

:::

### Casing

::: tip ✔ DO case two-letter acronyms with the same case.

- Right: `ioLimit` or `totalIOAmount`
- Wrong: `iOLimit` or `totalIoAmount`

:::

::: tip ✔ DO case three+ letter acronyms the same as a normal word.

- Right: `fidoKey` or `oauthUrl`
- Wrong: `webHTML`

:::

::: danger ✖ DO NOT capitalize the word following a <a href="https://www.thoughtco.com/common-prefixes-in-english-1692724">prefix</a> or words within a <a href="http://www.learningdifferences.com/Main%20Page/Topics/Compound%20Word%20Lists/Compound_Word_%20Lists_complete.htm">compound word</a>.

- Right: `subcategory`, `geocoordinate` or `crosswalk`
- Wrong: `metaData`, `semiCircle` or `airPlane`

:::

::: tip ✔ DO capitalize within hyphenated and open (spaced) compound words.

- Right: `fiveYearOld`, `daughterInLaw` or `postOffice`
- Wrong: `paperclip`, `changingroom` or `fullmoon`

:::

### Prefixes and Suffixes

::: tip ✔ DO suffix date and time properties.

- Right: `dueDate`&mdash;an `Edm.Date`
- Right: `createdDateTime`&mdash;an `Edm.DateTimeOffset`
- Right: `recurringMeetingTime`&mdash;an `Edm.TimeOfDay`
- Wrong: `dueOn` or `startTime`, both an `Edm.DateTimeOffset`

:::

::: danger ✖ DO NOT suffix property names with primitive type names unless the type is temporal.

- Right: `isEnabled` or `amount`
- Wrong: `enabledBool`

:::

::: tip ✔ DO prefix property names for properties concerning a different entity.

- Right: `siteWebUrl` on `driveItem`, or `userId` on `auditActor`
- Wrong: `webUrl` on `contact` when its the `companyWebUrl`

:::

### Common property names

| Approved name          | Type           | Use                                                       |
| ---------------------- | -------------- | --------------------------------------------------------- |
| `displayName`          | String         | A label that can be displayed or read aloud. Not `name`.  |
| `webUrl`               | String         | The web page for viewing or editing this entity.          |
| `url`                  | String         | A URL to a resource. (In Graph often holds the `webUrl`.) |
| `lastModifiedDateTime` | DateTimeOffset | The last time this entity changed.                        |
| `createdDateTime`      | DateTimeOffset | The time this entity was created.                         |
| `createdBy`            | identitySet    | The creator of this entity.                               |
| `createdByUser`        | user           | The user in `/users` who created this entity.             |



## Modelling Patterns

### Adding new subtypes

Table of Contents

- [Overview](#overview)
- [Background](#background)
- [Risks](#risks)
- [Mitigation](#mitigation)
- [Shielding clients](#shielding-clients)

This article discusses the consequences of introducing a new sub-type in the Microsoft Graph schema for a type that are used in collections.

A frequent pattern in Microsoft Graph is to have a small type hierarchy, a base type with a few subtypes (see [modeling variants](modeling-variants.md)). This allows to model collections of objects that have slightly different behavior. The common behavior is represented in the base type and the variations in a subtype, a concept very familiar from OO programming languages. It is straightforward to add a new subtype to the hierarchy with some consequences to the backwards compatibility as shown below.

#### Overview

OData allows to design collections of entities (entity sets, multi valued navigation properties) with values of different types. Currently these different types have to be subtypes of a common base type (often an abstract type). In the current version of Microsoft Graph are many collections of items that represent slightly different things, variants of one concept. For example, the [managedAppPolicy](https://docs.microsoft.com/en-us/graph/api/resources/intune-mam-managedapppolicy?view=graph-rest-1.0) type represents a base type for a variety of platform specific policies including for example a windowsInformationProtection policy. And when sending a GET request to the URL `/deviceAppManagement/managedAppPolicies` a collection of a mix of the sub-types is returned.
Most prominent are the collections of type directoryObject, an abstract base entity type that is implemented by types like user, groups, devices, etc..

Even though OData has means to express these subtypes and adding new subtypes is syntactically a backwards compatible change, there are situations that impose some risk to break client applications.

This article discusses the steps to ensure backwards compatibility of adding new subtypes that are potentially returned in collections.

#### Background

For heterogeneous collections, OData ensures that the client is able to distinguish the different types of the element of an collections. So for example querying

```HTTP
GET https://graph.microsoft.com/v1.0/groups/02bd9fd6-8f93-4758-87c3-1fb73740a315/owners
```

returns a collection where each element has an additional property `@odata.type`

```JSON
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#directoryObjects",
    "value": [
        {
            "@odata.type": "#microsoft.graph.user",
            "id": "48d31887-5fad-4d73-a9f5-3c356e68a038",
            "userPrincipalName": "MeganB@M365x214355.onmicrosoft.com"
            // ...
        }
    ]
}
```

Using the `@odata.type` property, the client code can decide how to deserialize the code, for example deciding the class used to create an object/instance.

Clients must anticipate that new subtypes get introduced and write code to guard against these situations. If not handled appropriately, the existing types most likely don't have the properties to store the returned JSON properties and these values have to be dropped. At the same time properties of the existing types can't get a value assigned. But even in untyped client code, without some compensation, there is no code that "looks" at the unexpected properties and misses the expected ones.

How to guard against and handle these situations is very specific to the client application and requires to understand the intended semantics of the types returned by the service.

#### Risks

There are a few potential risks when new sub-types are introduced. They are all variants of a) the fact that syntactically, previously expected properties might be absent and the received properties are ignored and b) the semantic role that the existing and new subtype play are potentially changing.

- De-serialization code might break because of missing properties in returned collection items. Even though property X was mandatory on all subtypes previously returned, the new subtype might not have this property and the client code needs to deal with that.

- Client libraries for strongly typed language might ignore some of the values in the @odata.type property without further configuration and need to be configured to be able to pick the right (client) type to deserialize into.

- In the client code it is reasonable to assume, based on existing running code, the "world" is exclusively described by the current subtypes. E.g. there is an understanding in Microsoft Graph's directory workload that there are two types of actors: users and servicePrincipals. Introducing a new subtype that can be an actor in directory (e.g. a device) would require a lot of clients to change to anticipate the presence of an object of that type and deal with it's own set of properties. In essence: originally each object was one of n types. After the change, an object can be neither of the n (because it is of the n+1's type).

#### Mitigation

Following are some of the techniques to mitigate these situation.

##### Avoid overgeneralized base types

If the abstract base type has many subtypes, it is quite likely that specific collection only ever contains a few subtypes. Since the type hierarchy is wide, there is probably some functionality or behavior that is only shared amongst a few but not all subtypes. That is ultimately explaining why some collection only contains some of the subtypes, the ones that share some behavior.

A well-known example for this situation is the `directoryObject` type which has many sub-types and only one property, `id`. Collections like for example the `owners` property on a `group` is declared as `directoryObject` and in reality only `user`s and `servicePrincipal`s are added to this collection since these are informally the only actors modeled in directory.

If one only focuses on the hierarchy, one could easily think it is straightforward to add a new subtype. What is necessary is the ensure that adding a new subtype doesn't change the semantic of the type hierarchy and the semantic of the property with it's implicit constraints.

##### Roll-out sequence

Microsoft Graph does not return object from a workload that has a type that is not configured in current metadata. That leads to behaviors that is slightly different depending if the object is returned as part of a collection or is requested individually.

If an object of an un-configured type is returned by the workload as part of a collection, the object just gets excluded from the collection and not returned. Microsoft Graph just doesn't know yet how to serialize the object.

If an object of an un-configured type is requested directly via an entity set, for example in case of `directoryObject` and a request like `/v1.0/directoryObject/{guid}`, Microsoft Graph returns an empty method body (not a 404 Not Found). It is assumed that the object and it's Id can not be found anywhere in the system and it is safe to ignore it. And Microsoft Graph just doesn't know yet how to serialize the object.

To make sure that the users do not get exposed to the second behavior, ensure that newly introduced entities are first known by graph before they get added to the collections.
The configuration allows Microsoft Graph to respond with the details/properties of the entities. Without that configuration Microsoft Graph returns no response body.
This leads necessarily to a two-step process of first introducing the entity type but not return them in any of the heterogeneous collections. And only after that returning them as items of collections. This can of be done in relatively rapid succession.

This is often not a problem since for utterly new entity types, no collection every has items of that type. But if the workload has APIs beside Microsoft Graph, these entities might have been added to the collections through that API.

##### Allow time for testing

Inform the clients about the change and allow them to test the changes in beta. Time is required implement the code necessary to deal with the new entity type, both in terms of de-serialization as well as integrating it into the rest of the application.

##### Communicate the change in semantics

Even more importantly, it is necessary for the client developers to incorporate the new semantic into their application/service, even if the change is perceived small. The addition of new data needs design changes in the client application. These changes potentially ripple through many layers of that application/service. This requires early communication and clear documentation what the new type represents and why/how it is considered a subtype of the original abstract type of the collection. Without that information the client application will not be able to process that data returned in the responses.

For example lets assume a situation where owners of a group are people and the only type ever returned as a member of the `owners` property of the `group` entity type is of type `user`. It is reasonable to assume that certain behaviors/functionality exists for these members of the owners collection. For example, every owner has an email address, every owner has a manager. By introducing new types of items to this collection, these assumption might not be true anymore (e.g. owners can be machine accounts without a manager or email). Even though the protocol and client libraries have ways to deal with the transport and de-serialization of these new types, it requires some new design how downstream modules of the client applications/services deal with entities that don't have email addresses or managers.

#### Shielding Clients

[TODO: describe upcoming features in Microsoft Graph to ensure full backwards compatibility]


### Entity Types and Complex Types

The [OData standard](http://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html), beside many other things, provides a way to describe the structure of the requests and responses of an OData service via the [Common Schema Definition Language](http://docs.oasis-open.org/odata/odata-csdl-xml/v4.01/odata-csdl-xml-v4.01.html) (CSDL).

CSDL defines a few ways to define [types](https://en.wikipedia.org/wiki/Data_type) and the most prominent are Entity Types and Complex Types. These two have some similarities and some differences that we are going to explore in this article.

#### Entity Types

Entity Types are that most common way to define the structure of the requests and responses of an OData service. The standard [defines](http://docs.oasis-open.org/odata/odata-csdl-xml/v4.01/odata-csdl-xml-v4.01.html#sec_EntityType):

> Entity Types are nominal structured types with a key that consists of one or more references to structural properties. An entity type is the template for an entity: any uniquely identifiable record such as a customer or order..

There is three things to note in that (arguably terse) definition:

- By "structured" the standard means that an Entity Type is defined by enumerating its (typed) **properties**.
- "nominal" just refers to the fact that the type **has a name** (a name in the CSDL schema).
- And the most important piece in the context of this article is that an Entity Type declares a **key property** and the consequence is that objects of this type can be uniquely identified through this key. In Microsoft Graph that key is currently always the property named "id". The standards allows more variation and Microsoft Graph might also relax this constraint in the future.

How they key is used to identify an object is a bit out of the scope of this document. For now it should suffice to say, that it is used as part of the URL to "name" an individual object. For Example in the URL https://localhost/api/authors/50 (or https://localhost/api/authors(50) ), the 50 is the key of an object.

#### Complex Types

Complex Types are non-scalar properties of entity types that enable scalar properties to be organized within entities. Complex types consist of a list of properties with no key, and can therefore only exist as properties of a containing entity. You can use complex types to group fields together without exposing them as an independent OData entity. Complex types can contain complex types, that is, they can be deeply nested.

- A complex type doesn't have keys and therefore cannot exist independently.
- Complex type can only exist as properties of entity types or other complex types.
- It cannot participate in relationships (see navigation properties) directly.

#### Comparison

In the example below, we have added an Author as an Entity Type and Address a Complex Type.

```XML
      <EntityType Name="Author">
        <Key>
          <PropertyRef Name="id" />
        </Key>
        <Property Name="id" Type="Edm.Int32" Nullable="false" />
        <Property Name="name" Type="Edm.String" />
        <Property Name="address" Type="microsoft.graph.Address" />
      </EntityType>
      <ComplexType Name="Address">
        <Property Name="city" Type="Edm.String" />
        <Property Name="street" Type="Edm.String" />
        <Property Name="stateOrProvince" Type="Edm.String" />
        <Property Name="country" Type="Edm.String" />
      </ComplexType>
```

You can see that Address type does not have any sort of key property. Complex types cannot be tracked on their own, so as a property in the Author class, it will be **tracked as part of** an author object. The consequence is that its lifecycle is coupled to the enclosing Entity Type: When the author gets deleted, the address gets deleted as well.

#### Summary

In Summary:

- Both Entity Types and Complex types are named types that declare a list of properties for the objects of that type.
- An Entity Type always has a key declared whereas a Complex type doesn't.
- Objects of an Entity Type can be directly addressed via an URL but ComplexTypes are always contained in an EntityType object and can only be addressed through a combination of an Entity address and a property name. (see more at [Navigation Properties and Containment](navigation-containment.md))

### Adding Members to Enumerations

Microsoft Graph services sometimes want to add a member to an enumeration type. However, there are barriers. First and foremost, some deserializers (including Json.NET) fail if an enumeration property has a value not found in that property's enumeration type. Second, a client may not deal with enumeration values unknown to it. The `Evolvable Enumerations` pattern and implementation allows a member to be safely be added to an enumeration.

#### Evolvable Enumerations

An evolvable enumeration contains the sentinel member `unknownFutureValue` after which new enumeration members are added. Consider the following enumeration:

```xml
<EnumType Name="weekday">
  <EnumMember Name="monday"/>
  <EnumMember Name="tuesday"/>
  ...
  <EnumMember Name="sunday"/>
  <EnumMember Name="unknownFutureValue"/>
</EnumType>
```

From this the C# SDK generates:

```csharp
public enum weekday
{
  monday,
  tuesday,
  ...
  sunday,
  unknownFutureValue
}
```

The new enumeration member `newday` is added after `unknownFutureValue`:

```xml
<EnumType Name="weekday">
  <EnumMember Name="monday"/>
  <EnumMember Name="tuesday"/>
  ...
  <EnumMember Name="sunday"/>
  <EnumMember Name="unknownFutureValue"/>
  <EnumMember Name="newday"/>     <!-- new value -->
</EnumType>
```

From this the C# SDK generates:

```csharp
public enum weekday
{
  monday,
  tuesday,
  ...
  sunday,
  unknownFutureValue,
  newday      // new value
}
```

#### Methods and client opt-in

On POST, if any enumeration property of the entity contains `unknownFutureValue`, the request will fail with `400 Bad Request`. On PATCH, any enumeration property with value `unknownFutureValue` is ignored--that property is not updated.

Callers signal their ability to process added members by including the `include-unknown-enum-members` preference:

```http
GET /me/calendar
Prefer: include-unknown-enum-members
```

Upon GET, when this header is absent, `unknownFutureValue` is returned to the caller for enumeration property values that are one of the added enumeration members. When this header is present, the enumeration value is returned unchanged.

Upon a filtered GET, when this header is absent, if `unknownFutureValue` appears in a `$filter` clause it matches any added enumeration member. For example, if the enumeration members `newday` and `anotherNewDay` have been added to `weekday`, these are equivalent:

```http
$filter=weekday eq unknownFutureValue
$filter=weekday eq newday or weekday eq anotherNewDay
$filter=weekday ge unknownFutureValue
```

If the header is absent, `$filter=weekday` **eq** `unknownFutureValue` matches any new enumeration value. If the header is present, that same filter matches nothing, while `$filter=weekday` **ge** `unknownFutureValue` matches any new enumeration value. (Note: the latter matches new enumeration values whether the header is present or not.)

#### SDK code generation

The SDK generates enumeration definitions from the current schema. Requests always include the `include-unknown-enum-members` header.

At runtime, since other members could have been added to the enumeration after code was generated, values unknown to the generated code are translated to `unknownFutureValue`. When the service sees an enumeration property with the value `unknownFutureValue`, it will ignore it and not update the property.

#### Resetting an Evolvable Enumeration

Upon a major version change, `unknownFutureValue` can be moved to the end of the enumeration, making known the previously unknown enumeration members.

```csharp
public enum weekday
{
  monday,
  ...
  sunday,
  newday,
  anotherNewDay,
  unknownFutureValue
}
```


## Common Patterns

## Advanced Patterns

## Design Rules