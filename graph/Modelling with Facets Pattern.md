# Facets Pattern

Microsoft Graph API Design Pattern

## *A frequent pattern in Microsoft Graph is to model multiple variants of a common concept as a single entity type with common properties and one facet property (of complex type) per variant.*

## Context

Let’s assume you need to create an API to manage documents, pictures, files of
different formats which are organized in different folder hierarchies across
multiple local and shared drives. These items have many common properties such
as Name, Owner, Creation Date, common relationships like activities and
subscriptions, and a common set of behaviors like CRUD operations and sharing.

There are also subsets of values that are specific for each variant, for example
hashes and Mime type for files and eight different properties for photo like
camera model and settings.

Since usually individual users or organizations deal with a vast amount of
information stored in files there is a need for easy filtering and querying
information based on its metadata.

While modeling for existing requirements we need to create a flexible API design
to be able accommodate future needs like new metadata or behavior.

## Problem

How to model files and folders as API resources to be able to easily mange them,
query and filter using metadata, and

A more general problem is how to model a collection of heterogeneous elements
that have a set of common properties and behaviors, and some unique properties
for each variant.

## Solution

OData allows us to design collections of entities (entity sets, multi valued
navigation properties) with values of different types using **type hierarchy**,
where there is one abstract base type with a few common properties and one
sub-type for each variant of the entity. In the current version of Microsoft
Graph there are many collections of items that represent slightly different
things, variants of one concept.

## Issues and Considerations

When introducing a new subtype, you need to ensure that the new subtype doesn't
change the semantic of the type hierarchy with it's implicit constraints.

There are a **few potential risks** for client applications when new sub-types
are introduced:

-   De-serialization code might break because of missing properties in returned
    collection items. Even though property X was mandatory on all subtypes
    previously returned, the new subtype might not have this property and the
    client code needs to deal with that.

-   Client libraries for strongly typed language might ignore some of the values
    in the @odata.type property without further configuration and need to be
    configured to be able to pick the right (client) type to deserialize into.

To minimize impact on clients type hierarchy can be refined by annotating the
collections with OData derived type constraints (see validation vocabulary).
This annotation restricts the values to certain sub-trees of an inheritance
hierarchy. It makes it very explicit that the collection only contains elements
of some of the subtypes and helps to not return objects of a type that is
semantically not suitable. In addition, you can follow some of the mitigation
techniques such as:

-   Avoid overgeneralized base types

-   Think about roll-out sequence

    -   Consider that Microsoft Graph does not return objects from a workload
        that has a type that is not configured in current metadata. To avoid
        inconsistencies, follow a two-step process:

    -   Introduce the entity type to the Graph metadata but don’t return objects
        of the type in any of the heterogeneous collections.

    -   Enable your workload to return objects of the new type as items of
        collection.

-   Allow time for testing

    -   Inform the clients about the change and allow them to test the changes
        in beta. Time is required to implement the code necessary to deal with
        the new entity type, both in terms of de-serialization as well as
        integrating it into the rest of the application.

-   Communicate the change in semantics

    -   It is necessary for the client developers to incorporate the new
        semantic into their application/service, even if the change is perceived
        to be small. This requires early communication and clear documentation
        of what the new type represents and why/how it is considered a subtype
        of the original abstract type of the collection.

## When to Use this Pattern

The Type hierarchy pattern is well familiar to OOP developers and well suited
for strongly typed client programming languages.

There are related patterns to consider such as
[Facets](https://github.com/microsoft/api-guidelines/tree/graph/graph) and [Flat
bag of
properties](https://github.com/microsoft/api-guidelines/tree/graph/graph).

## Example

`GET
<https://graph.microsoft.com/v1.0/groups/02bd9fd6-8f93-4758-87c3-1fb73740a315/owners>
`
returns a collection where each element can be a user or a service principal,
and has an additional property @odata.type to show subtype for each variant:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{
    "@odata.context":
"https://graph.microsoft.com/v1.0/\$metadata\#directoryObjects",
    "value": [
        {
            "@**odata.type**": "\#**microsoft.graph.user**",
            "id": "48d31887-5fad-4d73-a9f5-3c356e68a038",
            "userPrincipalName": "MeganB@M365x214355.onmicrosoft.com"
            // ...
        }
    ]
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
