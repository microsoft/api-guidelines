# Type Hierarchy

Microsoft Graph API Design Pattern


### *A frequent pattern in Microsoft Graph is to have a small type hierarchy, a base type with a few subtypes. This allows us to model collections of objects that have slightly different metadata and behavior.*

## Problem

The API design requires to model a set of entities based on a common concept
that can be further grouped into **mutually exclusive variants** with specific
properties and behaviors. The API design should be evolvable and allow addition
of new variants without breaking changes.

## Solution

API designers can use OData **type hierarchy**, where there is one abstract base
type with a few shared properties representing the common concept and one
sub-type for each variant of the entity. In hierarchy, the interdependencies of properties, i.e. which properties are relevant for which variants, is fully captured in metadata and client code can potentially leverage that to construct and/or validate requests.

## Issues and Considerations

When introducing a new subtype to the hierarchy, developers need to ensure that
the new subtype doesn't change the semantic of the type hierarchy with its
implicit constraints.
To retrieve properties specific for a derived type an API request URL may need to include casting to the derived type. If type hierarchy is very deep then resulting URL may become very long and not easily readable.  

There are a few consideration to take into account when new sub-types
are introduced:

-  TODO add something about SDK dependencies and required actions
-  TODO Client libraries for strongly typed language might ignore some of the values
    in the @odata.type property without further configuration and need to be
    updated to be able to pick the right (client) type to deserialize into.
-  In the case of public APIs in GA versions clients may develop their applications to support exclusively the current set of subtypes and don’t expect new variations. To mitigate the risk of clients disruption, when introducing a new subtype, allow ample time for communication and rollout.



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
and groups stored in Azure Active Directory. Since any a directoryObject object is a unique entity, the directoryObject type itself is derived from the graph.entity base type.

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

Response payload shortened for readability:

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
        ...        
    ]
}
```
API request for a subtype specific property requires type casting to the subtype, i.e. to retrieve jobTitle property, enabled for the user type, you need to cast from the directoryObject collection items to the microsoft.graph.group derived type.

```
GET https://graph.microsoft.com/v1.0/groups/a94a666e-0367-412e-b96e-54d28b73b2db/members/microsoft.graph.user?$select=displayName,jobTitle

Response payload shortened for readability:

{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users(displayName,jobTitle)",
    "value": [
        {
            "displayName": "John Cob",
            "jobTitle": "RESEARCHER II"
        },
       ...
    ]
}
```