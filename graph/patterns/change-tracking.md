# Change Tracking

Microsoft Graph API Design Pattern

*The change tracking pattern provides the ability to keep a third party system in sync with changes in Microsoft Graph without having to continuously poll the API.*

## Problem
---------

Third party systems require to keep data in sync with Microsoft Graph, and the API design should not allow for continuous polling as it'd be costly and wouldn't guarantee data integrity.

## Solution
--------

API designers leverage the change tracking (delta) capability on the entity set and declare a delta function for API consumers to use when tracking changes happening in the system.

This new endpoint can be uses to sync third party systems. This is achieved through returning a delta link with a watermark. Once the third party system comes back online, it uses the last provided delta link to catch up on new changes since their last request. Delta guarantees integrity of data through the watermark, regardless of service partitions and other obscure aspects for clients.

## Issues and Considerations
-------------------------

Implementer MUST implement a watermark storage system in case of active watermarks (cursor in data store, partition affinity, sync state in data store...).

## When to Use this Pattern
------------------------

Before using the change tracking pattern in your API definition, make sure your scenario fits the following criteria:

- API consumers want to sync the data.
- API consumers don't want to get notified of changes. (see change notifications pattern for this scenario).
- API consumers are not looking for a "one-time" export or back-up mechanism.

### Alternatives

- Change notifications pattern (TODO add link when described)
- Export pattern (TODO)
- Backup pattern (TODO)

## Examples
-------

### Getting changes for the users entity set

```HTTP
GET https://graph.microsoft.com/v1.0/users/delta

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
    ]
}
```

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
