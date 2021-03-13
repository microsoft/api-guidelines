---
title: Entity Types and Complex Types
---

# Entity Types and Complex Types

The [OData standard](http://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html), beside many other things, provides a way to describe the structure of the requests and responses of an OData service via the [Common Schema Definition Language](http://docs.oasis-open.org/odata/odata-csdl-xml/v4.01/odata-csdl-xml-v4.01.html) (CSDL).

CSDL defines a few ways to define [types](https://en.wikipedia.org/wiki/Data_type) and the most prominent are Entity Types and Complex Types. These two have some similarities and some differences that we are going to explore in this article.

## Entity Types

Entity Types are that most common way to define the structure of the requests and responses of an OData service. The standard [defines](http://docs.oasis-open.org/odata/odata-csdl-xml/v4.01/odata-csdl-xml-v4.01.html#sec_EntityType):

> Entity Types are nominal structured types with a key that consists of one or more references to structural properties. An entity type is the template for an entity: any uniquely identifiable record such as a customer or order..

There is three things to note in that (arguably terse) definition:

- By "structured" the standard means that an Entity Type is defined by enumerating its (typed) **properties**.
- "nominal" just refers to the fact that the type **has a name** (a name in the CSDL schema).
- And the most important piece in the context of this article is that an Entity Type declares a **key property** and the consequence is that objects of this type can be uniquely identified through this key. In Microsoft Graph that key is currently always the property named "id". The standards allows more variation and Microsoft Graph might also relax this constraint in the future.

How they key is used to identify an object is a bit out of the scope of this document. For now it should suffice to say, that it is used as part of the URL to "name" an individual object. For Example in the URL https://localhost/api/authors/50 (or https://localhost/api/authors(50) ), the 50 is the key of an object.

## Complex Types

Complex Types are non-scalar properties of entity types that enable scalar properties to be organized within entities. Complex types consist of a list of properties with no key, and can therefore only exist as properties of a containing entity. You can use complex types to group fields together without exposing them as an independent OData entity. Complex types can contain complex types, that is, they can be deeply nested.

- A complex type doesn't have keys and therefore cannot exist independently.
- Complex type can only exist as properties of entity types or other complex types.
- It cannot participate in relationships (see navigation properties) directly.

## Comparison

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

## Summary

In Summary:

- Both Entity Types and Complex types are named types that declare a list of properties for the objects of that type.
- An Entity Type always has a key declared whereas a Complex type doesn't.
- Objects of an Entity Type can be directly addressed via an URL but ComplexTypes are always contained in an EntityType object and can only be addressed through a combination of an Entity address and a property name. (see more at [Navigation Properties and Containment](navigation-containment.md))
