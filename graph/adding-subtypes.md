---
title: Adding subtypes
owner: chrispre
---

# Adding new subtypes

Table of Contents

- [Overview](#overview)
- [Background](#background)
- [Risks](#risks)
- [Mitigation](#mitigation)
- [Shielding clients](#shielding-clients)

This article discusses the consequences of introducing a new sub-type in the Microsoft Graph schema for a type that are used in collections.

A frequent pattern in Microsoft Graph is to have a small type hierarchy, a base type with a few subtypes (see [modeling variants](Modeling-variants)). This allows to model collections of objects that have slightly different behavior. The common behavior is represented in the base type and the variations in a subtype, a concept very familiar from OO programming languages. It is straightforward to add a new subtype to the hierarchy with some consequences to the backwards compatibility as shown below.

## Overview

OData allows to design collections of entities (entity sets, multi valued navigation properties) with values of different types. Currently these different types have to be subtypes of a common base type (often an abstract type). In the current version of Microsoft Graph are many collections of items that represent slightly different things, variants of one concept. For example, the [managedAppPolicy](https://docs.microsoft.com/en-us/graph/api/resources/intune-mam-managedapppolicy?view=graph-rest-1.0) type represents a base type for a variety of platform specific policies including for example a windowsInformationProtection policy. And when sending a GET request to the URL `/deviceAppManagement/managedAppPolicies` a collection of a mix of the sub-types is returned.
Most prominent are the collections of type directoryObject, an abstract base entity type that is implemented by types like user, groups, devices, etc..

Even though OData has means to express these subtypes and adding new subtypes is syntactically a backwards compatible change, there are situations that impose some risk to break client applications.

This article discusses the steps to ensure backwards compatibility of adding new subtypes that are potentially returned in collections.

## Background

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

## Risks

There are a few potential risks when new sub-types are introduced. They are all variants of a) the fact that syntactically, previously expected properties might be absent and the received properties are ignored and b) the semantic role that the existing and new subtype play are potentially changing.

- De-serialization code might break because of missing properties in returned collection items. Even though property X was mandatory on all subtypes previously returned, the new subtype might not have this property and the client code needs to deal with that.

- Client libraries for strongly typed language might ignore some of the values in the @odata.type property without further configuration and need to be configured to be able to pick the right (client) type to deserialize into.

- In the client code it is reasonable to assume, based on existing running code, the "world" is exclusively described by the current subtypes. E.g. there is an understanding in Microsoft Graph's directory workload that there are two types of actors: users and servicePrincipals. Introducing a new subtype that can be an actor in directory (e.g. a device) would require a lot of clients to change to anticipate the presence of an object of that type and deal with it's own set of properties. In essence: originally each object was one of n types. After the change, an object can be neither of the n (because it is of the n+1's type).

## Mitigation

Following are some of the techniques to mitigate these situation.

### Avoid overgeneralized base types

If the abstract base type has many subtypes, it is quite likely that specific collection only ever contains a few subtypes. Since the type hierarchy is wide, there is probably some functionality or behavior that is only shared amongst a few but not all subtypes. That is ultimately explaining why some collection only contains some of the subtypes, the ones that share some behavior.

A well-known example for this situation is the `directoryObject` type which has many sub-types and only one property, `id`. Collections like for example the `owners` property on a `group` is declared as `directoryObject` and in reality only `user`s and `servicePrincipal`s are added to this collection since these are informally the only actors modeled in directory.

If one only focuses on the hierarchy, one could easily think it is straightforward to add a new subtype. What is necessary is the ensure that adding a new subtype doesn't change the semantic of the type hierarchy and the semantic of the property with it's implicit constraints.

### Roll-out sequence

Microsoft Graph does not return object from a workload that has a type that is not configured in current metadata. That leads to behaviors that is slightly different depending if the object is returned as part of a collection or is requested individually.

If an object of an un-configured type is returned by the workload as part of a collection, the object just gets excluded from the collection and not returned. Microsoft Graph just doesn't know yet how to serialize the object.

If an object of an un-configured type is requested directly via an entity set, for example in case of `directoryObject` and a request like `/v1.0/directoryObject/{guid}`, Microsoft Graph returns an empty method body (not a 404 Not Found). It is assumed that the object and it's Id can not be found anywhere in the system and it is safe to ignore it. And Microsoft Graph just doesn't know yet how to serialize the object.

To make sure that the users do not get exposed to the second behavior, ensure that newly introduced entities are first known by graph before they get added to the collections.
The configuration allows Microsoft Graph to respond with the details/properties of the entities. Without that configuration Microsoft Graph returns no response body.
This leads necessarily to a two-step process of first introducing the entity type but not return them in any of the heterogeneous collections. And only after that returning them as items of collections. This can of be done in relatively rapid succession.

This is often not a problem since for utterly new entity types, no collection every has items of that type. But if the workload has APIs beside Microsoft Graph, these entities might have been added to the collections through that API.

### Allow time for testing

Inform the clients about the change and allow them to test the changes in beta. Time is required implement the code necessary to deal with the new entity type, both in terms of de-serialization as well as integrating it into the rest of the application.

### Communicate the change in semantics

Even more importantly, it is necessary for the client developers to incorporate the new semantic into their application/service, even if the change is perceived small. The addition of new data needs design changes in the client application. These changes potentially ripple through many layers of that application/service. This requires early communication and clear documentation what the new type represents and why/how it is considered a subtype of the original abstract type of the collection. Without that information the client application will not be able to process that data returned in the responses.

For example lets assume a situation where owners of a group are people and the only type ever returned as a member of the `owners` property of the `group` entity type is of type `user`. It is reasonable to assume that certain behaviors/functionality exists for these members of the owners collection. For example, every owner has an email address, every owner has a manager. By introducing new types of items to this collection, these assumption might not be true anymore (e.g. owners can be machine accounts without a manager or email). Even though the protocol and client libraries have ways to deal with the transport and de-serialization of these new types, it requires some new design how downstream modules of the client applications/services deal with entities that don't have email addresses or managers.

## Shielding Clients

[TODO: describe upcoming features in Microsoft Graph to ensure full backwards compatibility]
