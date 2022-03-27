# Change Tracking

Microsoft Graph API Design Pattern

*The change tracking pattern provides the ability to keep API consumers in sync with changes in Microsoft Graph without having to continuously poll the API.*

## Problem
---------

API consumers require an efficient way to keep data in sync with Microsoft Graph, and the API design should be optimized to avoid polling as it is costly for consumers and producers alike as well as wouldn't guarantee data integrity.

## Solution
--------

API designers can enable the change tracking (delta) capability on entity collections by declaring a delta function for API consumers to use to track changes in that collection.

This new endpoint can be used to sync API consumers. This is achieved through returning a delta link with a watermark. Once the API consumer needs to refresh the data it uses the last provided delta link to catch up on new changes since their last request. Delta guarantees integrity of data through the watermark, regardless of service partitions and other obscure aspects for clients.

> Note: although this capability is similar to the [OData $delta feed](https://docs.oasis-open.org/odata/odata-json-format/v4.0/errata02/os/odata-json-format-v4.0-errata02-os-complete.html#_Toc403940644) capability, it is a different construct. Microsoft Graph APIs MUST provide change tracking through the delta function and MUST NOT implement the OData $delta feed when providing change tracking capabilities to ensure the uniformity of the API experience.

## Issues and Considerations
-------------------------

Implementer MUST implement a watermark storage system in case of active watermarks. Passive watermarks are watermarks that can be retrieved from the context (e.g. timestamp), active watermarks represent information required to track the sync state which cannot be retrieved from the context (e.g. cursor from data store, partition affinity marker, partition id, generated unique sync identifier...)

Implementer MUST implement soft deletion for entities in the backend storage system. The soft deletion will provide useful information to the client to appropriately reflect deletions.

When an entity is soft deleted, the delta function MUST return the id of the deleted entity as well as a `@removed` annotation with the `reason` field.
- The reason MUST be set to `changed` if the entity can be restored. `"@removed": {"reason": "changed"}`. 
- The reason MUST be set to `deleted` if the entity cannot be restored. `"@removed": {"reason": "deleted"}`.

When a link to an entity is deleted, or when the linked entity is deleted, or when a link to an entity is added, implementer MUST return a `property@delta` annotation. e.g. considering the entity Group has a navigation property named members of type Collection(user):

- When a user is added to the group `"members@delta": [{ "@odata.type": "#microsoft.graph.user", "id of the added user"}]`
- When a user is removed from the group, or the target user is deleted `"members@delta": [{"@removed": {"reason": "deleted"}, "id of the deleted or removed user"}]`

> Note: the delta function also provides support for $filter and $select to allow the API consumer to narrow down the number of entities and properties retrieved as well as the number of changes that are tracked. Additionally the delta function can also support $top to allow the API consumer to sync smaller sets of changes as well as $expand to allow the API consumer to sync related data. Expand across workloads is not supported today however.

## When to Use this Pattern
------------------------

Before using the change tracking pattern in your API definition, make sure your scenario fits the following criteria:

- API consumers want to sync the data.
- API consumers don't want to be immediately notified of changes. (see change notifications pattern for this scenario).
- API consumers are not looking for a "one-time" export or back-up mechanism.

### Alternatives

- Change notifications pattern (TODO add link when described)
- Backup pattern (TODO)

## Examples
-------

### Getting changes for the users entity set

```HTTP
GET https://graph.microsoft.com/v1.0/users/delta
```

```json
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users",
    "@odata.deltaLink": "https://graph.microsoft.com/v1.0/users/delta?$deltatoken=mS5DuRZGjVL-abreviated",
    "value": [
        {
            "businessPhones": [
                "+1 309 555 0104"
            ],
            "displayName": "Grady Archie",
            "givenName": "Grady",
            "jobTitle": "Designer",
            "mail": "GradyA@contoso.onmicrosoft.com",
            "officeLocation": "19/2109",
            "preferredLanguage": "en-US",
            "surname": "Archie",
            "userPrincipalName": "GradyA@contoso.onmicrosoft.com",
            "id": "0baaae0f-b0b3-4645-867d-742d8fb669a2",
            "manager@delta": [
                {
                    "@odata.type": "#microsoft.graph.user",
                    "id": "99789584-a1e1-4232-90e5-866170e3d4e7"
                }
            ]
        },
        {
            "id": "0bbbbb0f-b0b3-4645-867d-742d8fb669a2",
            "@removed": {
                "reason": "changed"
            }
        }
    ]
}
```

> Note: the response contains an `@odata.deltaLink` instance annotation with the watermark only when all the changes are enumerated. If more changes need to be enumerated, the response instead contains an `@odata.nextLink` instance annotation the application can request right away to get the next page.

### CSDL example

```xml
<Function Name="delta" IsBound="true">
    <Parameter Name="bindingParameter" Type="Collection(Microsoft.DirectoryServices.user)" />
    <ReturnType Type="Collection(Microsoft.DirectoryServices.user)" />
</Function>

<EntitySet Name="users" EntityType="microsoft.graph.user">
    <Annotation Term="Org.OData.Capabilities.V1.ChangeTracking">
    <Record>
        <PropertyValue Property="Supported" Bool="true" />
    </Record>
    </Annotation>
</EntitySet>
```
