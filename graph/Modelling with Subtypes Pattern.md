# Type Hierarchy

Microsoft Graph API Design Pattern

“Not supposed to be precise but easy to understand”

### *A frequent pattern in Microsoft Graph is to have a small type hierarchy, a base type with a few subtypes. This allows us to model collections of objects that have slightly different metadata and behavior.*

## Problem

The API design requires to model a set of entities based on a common concept
that can be further grouped into mutually exclusive variants with specific
properties and behaviors. The API design should be evolvable and allow addition
of new variants without breaking changes.

## Solution

API designers can use OData **type hierarchy**, where there is one abstract base
type with a few shared properties representing the common concept and one
sub-type for each variant of the entity.

## Issues and Considerations

When introducing a new subtype to the hierarchy, developers need to ensure that
the new subtype doesn't change the semantic of the type hierarchy with its
implicit constraints.

There are a **few potential risks** for client applications when new sub-types
are introduced:

-   De-serialization code might break because of missing properties in returned
    collection items. Even though property X was mandatory on all subtypes
    previously returned, the new subtype might not have this property and the
    client code needs to deal with that.

-   Client libraries for strongly typed language might ignore some of the values
    in the @odata.type property without further configuration and need to be
    updated to be able to pick the right (client) type to deserialize into.

In addition, you can follow some of the mitigation techniques such as:

-   Think about gradual roll-out sequence

    -   Consider that Microsoft Graph does not return objects from a workload
        that has a type that is not configured in current metadata. To avoid
        inconsistencies, follow a two-step process:

        -   Introduce the entity type to the Graph metadata but don’t return
            objects of the type in any of the heterogeneous collections.

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

The Type hierarchy pattern is well suited to use case where each variant of a
common concept has unique properties and behaviors, no combinations of variants
is anticipated, API queries are managed programmatically with type casting.

There are related patterns to consider such as
[Facets](https://github.com/microsoft/api-guidelines/tree/graph/graph) and [Flat
bag of
properties](https://github.com/microsoft/api-guidelines/tree/graph/graph).

## Example

The directoryObject type is the main abstraction for many directory
types such as users, organizational contacts, devices, service principals
and groups stored in Azure Active Directory. Since any directoryObject object is a unique entity the directoryObject type itself is derived from the  graph.entity base type.

```XML
<EntityType Name="entity" Abstract="true">
    <Key>
        <PropertyRef Name="id" />
    </Key>
    <Property Name="id" Type="Edm.String" Nullable="false" />
</EntityType>
<EntityType *Name*="directoryObject" *BaseType*="graph.entity" />
    <Property *Name*="deletedDateTime" *Type*="Edm.DateTimeOffset" />
<EntityType/>
```


Groups and users are derived types and modeled as

```XML
 <EntityType Name="group" BaseType="graph.directoryObject" />
        <Property Name="accessType" Type="graph.groupAccessType"
…
</EntityType>
<EntityType Name="user" BaseType="graph.directoryObject">
        <Property Name="aboutMe" Type="Edm.String" />
…
</EntityType>
```

API request to get members of a group returns a heterogeneous collection of
users and groups where each element can be a user or a group, and has an
additional property @odata.type for a variant subtype:

```
GET https://graph.microsoft.com/v1.0/groups/a94a666e-0367-412e-b96e-54d28b73b2db/members?$select=id,displayName
…
Response payload:

{
     "@odata.context":
"https://graph.microsoft.com/v1.0/\$metadata\#directoryObjects",
    "value": [
        {           
            "@odata.type": "#microsoft.graph.user",
            "id": "37ca648a-a007-4eef-81d7-1127d9be34e8",
            "displayName": "John Cob"
        },
        {
            "@odata.type": "#microsoft.graph.group",
            "id": "45f25951-d04f-4c44-b9b0-2a79e915658d",
            "displayName": "Department 456"
        },
…        
    ]
}
```
