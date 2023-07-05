# "Draft" Entity

Microsoft Graph API Design Pattern

*A "draft" entity is one which can be used to temporarily store a subset of an entity that it is a draft of, avoiding validations that "real" instances of the entity would normally require.*

## Problem

There are situations when a client only has a portion of the data required to create an entity at a given time. 
In these situations, the client needs to store that partial data somewhere without requiring all of the data, and with allowing potentially invalid data to be present.

## Solution

The solution is to have a "draft" entity for the desired entity. 
This "draft" can then be used to store the partial data until all of the data is available, at which time the "draft" can be used to create a "real" entity instance. 
Doing this allows the storage of the partial data without compromising the integrity of the model for the "real" entity.

```xml
<Entity Name="someRealEntity">
  <Key>
    <PropertyRef Name="id" />
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false" />
  <Property Name="firstProp" Type="Edm.String" Nullable="false" />
  <Property Name="secondProp" Type="Edm.Int32" Nullable="false" />
  ...
</Entity>

<Entity Name="someRealEntityDraft">
  <Key>
    <PropertyRef Name="id" />
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false" />
  <Property Name="firstProp" Type="Edm.String" Nullable="true" />
  <Property Name="secondProp" Type="Edm.Int32" Nullable="true" />
  ...
</Entity>

<Action Name="realize" IsBound="true">
  <Parameter Name="bindingParameter" Type="self.someRealEntityDraft" Nullable="false" />
</Action>
```

## When to use this pattern

The "draft" entity pattern will be useful for workloads that back a UI and expose entities which the UI creates through the use of a multi-page process. 
It is also useful for any circumstances where the [builder pattern](https://en.wikipedia.org/wiki/Builder_pattern) is effective, as this pattern is effectively a specialization of the builder pattern. 

## Issues and considerations

This pattern should be avoided; instead, use some storage external to graph. 
Having entities that cannot be directly used by services because they are imcomplete and that have relaxed validation requirements effectively exposes an API that is equivalent to blob storage. 

This pattern *may* be used as a stop-gap if a workload does not yet have the infrastructure for external storage; these entities must be deprecated immediately since they are only a stop-gap.

## Example

### {1} Create a "real" instance with only partial data, getting an error

```HTTP
POST /someRealEntities
{
  "firstProp": "a value"
}

400 Bad Request
{
  "error": {
    "code": "badRequest",
    "message": "The 'secondProp' property must not be null."
  }
}
```

### {2} Create a "draft" instance with only partial data

```HTTP
POST /someRealEntityDrafts
{
  "firstProp": "a value"
}

201 Created
{
  "id": "{id}",
  "firstProp": "a value",
  "secondProp": null,
  ...
}
```

### {3} Try to realize the draft while it still only has partial data, getting an error

```HTTP
POST /someRealEntityDrafts/{id}/realize

400 Bad Request
{
  "error": {
    "code": "badRequest",
    "message": "The 'secondProp' property must not be null."
  }
}
```

### {4} Continue updating the draft with new data as the data becomes known

```HTTP
PATCH /someRealEntityDrafts/{id}
{
  "secondProp": 14
}

204 No Content
```

### {5} Realize the draft once all data has been provided

```HTTP
POST /someRealEntityDrafts/{id}/realize

204 No Content
Location: /someRealEntities/{id2}
```

### {6} Retrieve the realized instance

```HTTP
GET /someRealEntities/{id2}

{
  "id": "{id2}",
  "firstProp": "a value",
  "secondProp": 14,
  ...
}
```
