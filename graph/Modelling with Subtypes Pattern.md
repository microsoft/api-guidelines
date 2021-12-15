# Type Hierarchy 

Microsoft Graph API Design Pattern

Not supposed to be precise but easy to understand 

*A frequent pattern in Microsoft Graph is to have a small type hierarchy, a base type with a few subtypes. This allows us to model collections of objects that have slightly different metadata and behavior.*
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
<BR>


## Problem
--------
The API design requires to model a set of entities based on a common concept that can be further grouped into mutually exclusive variants with specific properties and behaviors. The API design should be evolvable and allow addition of new variants without breaking changes.




## Solution
--------

OData allows us to design collections of entities (entity sets, multi
valued navigation properties) with values of different types using
**type hierarchy**, where there is one abstract base type with a few
common properties and one sub-type for each variant of the entity. 

## Issues and Considerations
-------------------------

When introducing a new subtype, you need to ensure that the new subtype
doesn't change the semantic of the type hierarchy with it's implicit
constraints.

There are a **few potential risks** for client applications when new
sub-types are introduced:

- De-serialization code might break because of missing
properties in returned collection items. Even though property X was
mandatory on all subtypes previously returned, the new subtype might not
have this property and the client code needs to deal with that.

- Client libraries for strongly typed language might ignore some
of the values in the @odata.type property without further configuration
and need to be configured to be able to pick the right (client) type to
deserialize into.

To minimize impact on clients type hierarchy can be refined by
annotating the collections with OData derived type constraints (see
validation vocabulary). This annotation restricts the values to certain
sub-trees of an inheritance hierarchy. It makes it very explicit that
the collection only contains elements of some of the subtypes and helps
to not return objects of a type that is semantically not suitable. In
addition, you can follow some of the mitigation techniques such as:

- Avoid overgeneralized base types

- Think about roll-out sequence
  - Consider that Microsoft Graph does not return objects from a workload
that has a type that is not configured in current metadata. To avoid
inconsistencies, follow a two-step process:
  - Introduce the entity type to the Graph metadata but don’t
return objects of the type in any of the heterogeneous collections.
  - Enable your workload to return objects of the new type as items
of collection.


- Allow time for testing
  - Inform the clients about the change and allow them to test the
changes in beta. Time is required to implement the code necessary to
deal with the new entity type, both in terms of de-serialization as well
as integrating it into the rest of the application.

- Communicate the change in semantics

  - It is necessary for the client developers to incorporate the new
semantic into their application/service, even if the change is perceived
to be small. This requires early communication and clear documentation
of what the new type represents and why/how it is considered a subtype
of the original abstract type of the collection.

## When to Use this Pattern
------------------------

The Type hierarchy pattern is well familiar to OOP developers and well
suited for strongly typed client programming languages.

There are related patterns to consider such as
[Facets](https://github.com/microsoft/api-guidelines/tree/graph/graph)
and [Flat bag of
properties](https://github.com/microsoft/api-guidelines/tree/graph/graph).

## Example
-------
For example let’s assume you need to model an API to manage groups in an
organization, where employees can create groups and become owners of the
group by default. At the same time to support business processes some
groups may be created automatically by daemon applications using a
service principal account. In this case the service principle will
become the group owner. People and service principles have some common
and some unique properties such as both have unique identifiers and
credentials, but users will have additional properties such as email and
manager for example. Conversely a service principle won’t have a manager
assigned but may have an associated application identifier and a
description.
GET
[https://graph.microsoft.com/v1.0/groups/02bd9fd6-8f93-4758-87c3-1fb73740a315/owners](https://graph.microsoft.com/v1.0/groups/02bd9fd6-8f93-4758-87c3-1fb73740a315/owners) 
returns a collection where each element can be a user or a service
principal, and has an additional property @odata.type to show subtype
for each variant:
```
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
```
