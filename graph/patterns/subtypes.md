# Type hierarchy

Microsoft Graph API Design Pattern

*A frequent pattern in Microsoft Graph is to have a small type hierarchy, a base type with a few subtypes. This lets us model collections of resources that have slightly different properties and behavior.*

## Problem

The API design requires that we model a set of resources based on a common concept
that can be further grouped into *mutually exclusive variants* with specific
properties and behaviors. The API design should be evolvable and allow the addition
of new variants without breaking changes.

## Solution

API designers might use a *type hierarchy*, where there is one base
type (which might be abstract) with a few shared properties representing the common concept and one
subtype for each variant of the resource. In the hierarchy, the interdependencies of properties, that is, which properties are relevant for which variants, is fully captured in the type system.

## When to use this pattern

Use this pattern where each variant of a common concept has its own unique properties and behaviors,
no combination of variants is anticipated, and it is acceptable that callers who need to query resources by variant are adequately served by filtering or partitioning using type casting.

You can consider related patterns such as [facets](./facets.md) and [flat bag of properties](./flat-bag.md).

## Issues and considerations

When introducing a new subtype to the hierarchy, developers need to ensure that
the new subtype doesn't change the semantic of the type hierarchy or collections of the specified base type with implicit constraints.

To reference properties specific to a derived type, an API request URL might need to include a segment casting to the derived type. If the type hierarchy is very deep, then the resulting URL might become very long and not easily readable.  

There are a few considerations to take into account when new subtypes are introduced:

- *TODO add something about SDK dependencies and required actions*
- *TODO* Client libraries for a strongly typed language might ignore some of the values
    in the @odata.type property without further configuration and need to be
    updated to be able to pick the right (client) type to deserialize into.
- In the case of public APIs in GA versions, clients might develop their applications to support exclusively the current set of subtypes, and don’t expect new variations. To mitigate the risk of clients' disruption, when introducing a new subtype, allow ample time for communication and rollout.

## Example

The directoryObject type is the main abstraction for many directory
types such as users, organizational contacts, devices, service principals,
and groups stored in Azure Active Directory. Because any directoryObject object is a unique entity, the directoryObject type itself is derived from the `graph.entity` base type.

```XML
<EntityType Name="entity" Abstract="true">
    <Key>
        <PropertyRef Name="id" />
    </Key>
    <Property Name="id" Type="Edm.String" Nullable="false" />
</EntityType>
<EntityType Name="directoryObject" BaseType="graph.entity" />
    <Property Name="deletedDateTime" Type="Edm.DateTimeOffset" />
<EntityType/>
```

Groups and users are derived types and modeled as follows:

```XML
 <EntityType Name="group" BaseType="graph.directoryObject" />
   <Property Name="description" Type="Edm.String" />
   ...
</EntityType>
<EntityType Name="user" BaseType="graph.directoryObject">
   <Property Name="jobTitle" Type="Edm.String" />
   ...
</EntityType>
```

An API request to get members of a group returns a heterogeneous collection of
users and groups where each element can be a user or a group, and has an
additional `@odata.type` property that specifies the subtype:

```
GET https://graph.microsoft.com/v1.0/groups/a94a666e-0367-412e-b96e-54d28b73b2db/members

Response payload shortened for readability. The deletedDateTime property from the base type is a non-default property and is only returned if explicitly requested.

{
     "@odata.context":
"https://graph.microsoft.com/v1.0/$metadata#directoryObjects",
    "value": [
        { 
            "@odata.type": "#microsoft.graph.user",
            "id": "37ca648a-a007-4eef-81d7-1127d9be34e8",
            "jobTitle": "CEO",
            ...
        },
        {
            "@odata.type": "#microsoft.graph.group",
            "id": "45f25951-d04f-4c44-b9b0-2a79e915658d",
            "description": "Microsoft Graph API Reviewers",
            ...
        },
        ...        
    ]
}
```

Addressing a property of the subtype, for example, in `$filter` or `$select`, requires prefixing the property with the fully-qualified name of the subtype (or type derived from the subtype) on which it is defined. To filter on the `jobTitle` for the user type, you need to qualify the property with `microsoft.graph.user`. 

The following query returns all groups that are members of group a94a666e-0367-412e-b96e-54d28b73b2db, as well as users that are members and whose jobTitle is CEO.

```
GET https://graph.microsoft.com/v1.0/groups/a94a666e-0367-412e-b96e-54d28b73b2db/members?$filter=microsoft.graph.user/jobTitle eq 'CEO'

Response payload shortened for readability:

{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#directoryObjects",
    "value": [
        {
            "@odata.type": "#microsoft.graph.user",
            "id": "37ca648a-a007-4eef-81d7-1127d9be34e8",
            "jobTitle": "CEO",
            ...
        },
        {
            "@odata.type": "#microsoft.graph.group",
            "id": "45f25951-d04f-4c44-b9b0-2a79e915658d",
            "description": "Microsoft Graph API Reviewers",
            ...
        },
       ...
    ]
}
```

An entire collection can be cast to a particular subtype by appending the fully-qualified subtype name to the URL. Doing so filters the collection to members of (or derived from) that particular subtype, and makes the properties of that subtype available without casting. In this case, the `@odata.type` attribute is not returns for records of the specified subtype because the `@odata.context` indicates that the entire collection is consists of the particular subtype. Types derived from that subtype do still have the `@odata.type` attribute.

The following query returns only users that are members of group a94a666e-0367-412e-b96e-54d28b73b2db and whose jobTitle is CEO.

```
GET https://graph.microsoft.com/v1.0/groups/a94a666e-0367-412e-b96e-54d28b73b2db/members/microsoft.graph.user?$filter=jobTitle eq 'CEO'

Response payload shortened for readability:

{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users",
    "value": [
        {
            "id": "37ca648a-a007-4eef-81d7-1127d9be34e8",
            "jobTitle": "CEO",
            ...
        },
       ...
    ]
}
```

An API request to create a subtype object in a polymorphic collection requires "@odata.type" specified in the request body.

```
POST https://graph.microsoft.com/v1.0/directoryObjects

{
    "@odata.type": "#microsoft.graph.group",
    "description": "Microsoft Graph API Reviewers",
    ...
}
```
