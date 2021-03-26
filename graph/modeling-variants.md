---
title: Modeling variants
owner: chrispre
---

# Modeling Variants

Frequently we encounter situations where a certain piece of data in Microsoft Graph comes in different variants. Depending on the situation we call these variants, kinds, types, etc.. Some examples are

- owners of groups can be either a `user` or a `servicePrincipal`.
- an approver of a request can be a single user, a group, etc..
- the end of a recurring event can be after a number of repetitions or at a certain date, or never.

All these variants have different properties representing the information needed in these cases.

OData and Microsoft Graph offer different ways to model the API and these different variants. We'll describe those here, and list the advantages and disadvantages of each modeling technique.

In the remainder of the document we are using the term "variant" instead of "kind", "flavor", "type". "type" is defined by OData and we do not want to presume there has to be a type per variant.

## Approaches

There are different approaches to design a model in situations with multiple variants of common concept. We are going to compare three common patterns that we see in Microsoft Graph today.

### Type Hierarchy

A shallow **type hierarchy**: One abstract base type with a few common properties and one sub-type for each variant. OData adds `@odata.type` properties to the JSON representation when instances of these types are returned so that a client can quickly distinguish them.

One prominent example is the base type [graph.outlookItem](https://docs.microsoft.com/en-us/graph/api/resources/outlookitem?view=graph-rest-1.0) with subtypes like [message](https://docs.microsoft.com/en-us/graph/api/resources/message?view=graph-rest-1.0), [contact](https://docs.microsoft.com/en-us/graph/api/resources/contact?view=graph-rest-1.0), [event](https://docs.microsoft.com/en-us/graph/api/resources/event?view=graph-rest-1.0).

### Facets

A single entity type with **facets**: One type in the schema with common properties and one property (of complex type) per variant. The facet properties only have a value when the object represents that variant.

This can be seen for example in [driveItem](https://docs.microsoft.com/en-us/graph/api/resources/driveitem?view=graph-rest-1.0) where there are four variants (folder, file, image, photo) and one property per variant with the same name. These properties are modeled as a complex types that holds all information for that specific facet/variant. (e.g. just the `element count` property for folder and eight different properties for photo like camera model and settings).

### Flat

A **flat** bag of properties: One entity type with all the potential properties plus an additional property to distinguish the variants, often called `type`. The `type` property describes the variant and also defines properties are required/meaningful for the variant given by the `type` property.

Since the name `type` could be confused with the notion of type in OData, it is often recommended to qualify the property name. E.g. `recurrenceType` instead of `type`.

A good example for this is the recurrencePattern and recurrenceRange types (both properties on [patternedRecurrence](https://docs.microsoft.com/en-us/graph/api/resources/patternedrecurrence?view=graph-rest-1.0)).
The recurrencePattern has 6 variants expressed as 6 different values of the `type` property (e.g. daily, weekly, ...).
The key here is that for each of these values, some properties are meaningful and others are ignored. (e.g. `daysOfWeek` is relevant when `type` is `weekly` but not when it is `daily`).

## Pros and Cons

Below are a few pros and cons to decide which pattern to use.

- In **[hierarchy](#type-hierarchy)**, the interdependencies of properties, i.e. which properties are relevant for which variants, is fully captured in metadata and client code can potentially leverage that to construct and/or validate requests.
- Introducing new cases in **[hierarchy](#type-hierarchy)** is relatively isolated (which is why it is so familiar to OOP) and is considered backwards compatible (at least syntactically). But see the note about [changing semantics](#semantics) below.
- Introducing new cases/variants in **[facets](#facets)** is straightforward. One needs to be careful since it can introduce situations where previously exactly one of the facets was non-null and now all the old ones are null. For example imagine a new facet "shortcut" is added to the example above where everything was one of folder,file,image,photo. Adding the shortcut facet means that there are now object with all of the previous four are null.
  This is not unlike adding new subtypes in the hierarchy pattern or adding a new type value in the flat pattern.
- **[hierarchy](#type-hierarchy)** and **[facets](#facets)** (to a slightly lesser degree) are well suited for strongly typed client programming languages. Whereas **[flat](#flat)** is more familiar to developers of less strongly typed languages.
- **[facets](#facets)** has the potential to model what is typically associated with multiple inheritance (but it is not inheritance so please don’t quote me). Just to illustrate the point and constructing a highly hypothetical scenario, in the OneDrive example, having an item be a folder and a photo is easy to represent.
- **[facets](#facets)** and **[flat](#flat)** lend to syntactically simpler filter query expression. **[hierarchy](#type-hierarchy)** is more explicit but requires the less well known cast segments in the filter query. For example, if one wants to filter on the importance of a mail in a collection of outlookItems, one first needs to "cast" to mailItem to then filter on the importance property: `$filter=microsoft.graph.mailItem/importance eq 'High'`.
- **[flat](#flat)** might resemble a structure that developers are familiar with from on-prem products and their API (e.g. recurrence in Microsoft Graph is modeled after Exchange Server's model). Even though the Graph API can and should abstract from the implementation details this can have benefits in documentation and adoption.
- **[hierarchy](#type-hierarchy)** can become hard to maintain if the base type is quite abstract and the hierarchy is relatively wide. Lets assume a situation where collections are modeled using the base type with many sub-types, but the actual elements of the collection are only ever one or two of the sub-types. When a new subtype gets introduced and the collection(s) quickly contain elements of this new sub-type, client code has to react to these changes. It is important to check if this changes the semantics of the property (actual or assumed). See also [changing semantics](#semantics) below.
- Even though not frequently used in Microsoft Graph, **[hierarchy](#type-hierarchy)** can be refined by annotating the collections with OData `derived type constraints` (see [validation vocabulary](https://github.com/oasis-tcs/odata-vocabularies/blob/master/vocabularies/Org.OData.Validation.V1.md)). This annotation restricts the values to certain sub-trees of an inheritance hierarchy. It makes it very explicit that the collection only contains elements of some of the subtypes and helps to not return object of a type that is semantically not suitable.

## Future

The OData team is looking for feedback what is missing in terms of modeling tools and expressiveness that can help making these design decisions.

One of the options to explore that helps address some of the cons with overly broad subtype hierarchy is the OData annotation term `MayImplement` (see [Core vocabulary](https://github.com/oasis-tcs/odata-vocabularies/blob/master/vocabularies/Org.OData.Core.V1.md)), a feature that is not yet implemented in the OData libraries or Microsoft Graph.

The `MayImplement` annotation is defined as

> A collection of qualified type names outside of the type hierarchy that instances of this type might be addressable as by using a type-cast segment

This would allow to keep the type hierarchy narrow but still have some objects cast to a type outside that hierarchy. Please contact us to discuss if this could be helpful for your scenario.

## Summary

<a name="semantics"></a>As can be seen in a few of the Pros and Cons, one of the important aspects discussed here, is that the API design goes beyond the syntactical aspects of the API and it is important to plan ahead how the API evolves, lay the foundation, and allow the users to form a good understanding of the semantics of the API. **Changing the semantics is always a breaking change**. The different modeling patterns as described above, differ in how they express syntax and semantic and how they allow the API to evolve without breaking compatibility.
