# Upsert

Microsoft Graph API Design Pattern

*The `Upsert` pattern is a non-destructive idempotent operation using a client-provided key, that ensures that system resources can be deployed in a reliable, repeatable, and controlled way, typically used in Infrastructure as Code (IaC) scenarios.*

## Problem

Infrastructure as code (IaC) defines system resources and topologies in a declarative manner that allows teams to manage those resources as they would code.
Practicing IaC helps teams deploy system resources in a reliable, repeatable, and controlled way. 
IaC also helps automate deployment and reduces the risk of human error, especially for complex large environments. 
Customers want to adopt IaC practices for many of the resources managed through Microsoft Graph.

Most resources' creation operations in Microsoft Graph are not idempotent in nature.
As a consequence, API consumers that want to offer IaC solutions, must create compensation layers that can mimic idempotent behavior. 
For example, when creating a resource, the compensation layer must check whether the resource first exists, before trying to create or update the resource.

Additionally, IaC code scripts or templates usually employ client-provided names (or keys) to track resources in a predictable manner, whereas [Microsoft Graph guidelines](../GuidelinesGraph.md#behavior-modeling) suggests use of `POST` to create new entities with service-generated keys.

## Solution

The solution is to use an `Upsert` pattern, to solve for the non-idempotent creation and client-provided naming problems.

* For IaC scenarios, resources must use `Upsert` semantics with a client-provided key:
  * Use `PATCH` with a client-provided key:
    * If there is a natural client-provided key that can serve as the primary key, then the service should support `Upsert` with that key.
    * If the primary key is service-generated, the client-provided key should use an [alternate key](./alternate-key.md) to support idempotent creation.
  * For a non-existent resource (specified by the client-provided key) the service must handle this as a "create". As part of creation, the service must still generate the primary key value, if appropriate.
  * For an existing resource (specified by the client-provided key) the service must handle this as an "update".
  * Any new alternate key, used for IaC scenarios, should be called `uniqueName`, if there isn't already a more natural existing property that could be used as an alternate key.
* NOTE: the service must also support `GET` using the alternate key pattern.
* Services should always support `POST` to the collection URL.
  * For service-generated keys, this should return the server generated key.
  * For client-provided keys, the client should provide the key as part of the request payload.
* If a service does not support `Upsert`, then a `PATCH` call against a non-existent resource must result in an HTTP "409 conflict" error.

This solution allows for existing resources that follow Microsoft Graph conventions for CRUD operations to add `Upsert` without impacting existing apps or functionality.

Ideally, all new entity types should support an `Upsert` mechanism, especially where they support control-plane APIs, or are used in admin style or IaC scenarios.

## When to use this pattern

This pattern should be adopted for resources that are managed through infrastructure as code or desired state configuration.

## Issues and considerations

* The addition of this new pattern (with an alternate key) to an existing API does not represent a breaking change.
However, some API producers may have concerns about accidental usages of this new pattern unwittingly creating many new resources when the intent was an update.
As a result, API producers can use the `Prefer: idempotent` to require clients to opt-in to the Upsert behavior.
* The client-provided alternate key must be immutable after being set. If its value is null then it should be settable as a way to backfill existing resources for use in IaC scenarios.
* API producers could use `PUT` operations to create or update, but generally this approach is not recommended due to the destructive nature of `PUT`'s replace semantics.
* API producers may annotate entity sets, singletons and collections to indicate that entities can be "upserted". The example below shows this annotation for the `groups` entity set.  

```xml
<EntitySet Name="groups" EntityType="microsoft.graph.group">
  <Annotation Term="Org.OData.Capabilities.V1.UpdateRestrictions">
    <Record>
      <PropertyValue Property="Upsertable" Bool="true"/>
    </Record>
  </Annotation>
</EntitySet>
```

* `Upsert` can also be supported against singletons, using a `PATCH` to the singleton's URL.

## Examples

For these examples we'll use the `group` entity type, which defines both a primary (service-generated) key (`id`) and an alternate (client-provided) key (`uniqueName`).  

```xml
<EntityType Name="group">
  <Key>
    <PropertyRef Name="id"/>
  </Key>
  <Property Name="id" Type="Edm.String"/> 
  <Property Name="uniqueName" Type="Edm.String"/>
  <Property Name="displayName" Type="Edm.String"/>
  <Property Name="description" Type="Edm.String"/> 
  <Annotation Term="Org.OData.Core.V1.AlternateKeys">
    <Collection>
      <Record Type="Org.OData.Core.V1.AlternateKey">
        <PropertyValue Property="Key">
        <Collection>
            <Record Type="Org.OData.Core.V1.PropertyRef">
            <PropertyValue Property="Name" PropertyPath="uniqueName" />
            </Record>
        </Collection>
        </PropertyValue>
      </Record>
    </Collection>
  </Annotation>
  </Property>
</EntityType>
```

### Upserting a record (creation path)

Create a new group, with a `uniqueName` of "Group157". In this case, this group does not exist.

```http
PATCH /groups(uniqueName='Group157')
Prefer: idempotent; return=representation
```

```json
{
    "displayName": "My favorite group",
    "description": "All my favorite people in the world"
}
```

Response:

```http
201 created
Preference-Applied: idempotent; return=representation
```

```json
{
    "id": "1a89ade6-9f59-4fea-a139-23f84e3aef66",
    "displayName": "My favorite group",
    "description": "All my favorite people in the world",
    "uniqueName": "Group157"
}
```

### Upserting a record (update path)

Create a new group, with a `uniqueName` of "Group157", exactly like before. Except in this case, this group already exists. This is a common scenario in IaC, when a deployment template is re-run multiple times.

```http
PATCH /groups(uniqueName='Group157')
Prefer: idempotent; return=representation
```

```json
{
    "displayName": "My favorite group",
    "description": "All my favorite people in the world"
}
```

Response:

```http
200 ok
Preference-Applied: idempotent; return=representation
```

```json
{
    "id": "1a89ade6-9f59-4fea-a139-23f84e3aef66",
    "displayName": "My favorite group",
    "description": "All my favorite people in the world",
    "uniqueName": "Group157"
}
```

Notice how this operation is idempotent in nature, rather than returning a 409 conflict error.

### Updating a record

Update "Group157" group with a new description.

```http
PATCH /groups(uniqueName='Group157')
Prefer: idempotent; return=representation
```

```json
{
    "description": "Some of my favorite people in the world."
}
```

Response:

```http
200 ok
Preference-Applied: idempotent; return=representation
```

```json
{
    "id": "1a89ade6-9f59-4fea-a139-23f84e3aef66",
    "displayName": "My favorite group",
    "description": "Some of my favorite people in the world.",
    "uniqueName": "Group157"
}
```

### Upsert not supported

Create a new group, with a `uniqueName` of "Group157". In this case, this group does not exist and additionally
the service does not `Upsert` for groups.

```http
PATCH /groups(uniqueName='Group157')
Prefer: idempotent; return=representation
```

```json
{
    "displayName": "My favorite group",
    "description": "All my favorite people in the world"
}
```

Response:

```http
409 conflict
```
