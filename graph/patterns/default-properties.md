# Default properties

Microsoft Graph API Design Pattern

*The default properties pattern allows API producers to omit specific properties from the response unless they are explicitly requested using `$select`.*

## Problem

API designers want to control the set of properties that their entities return by default, when the incoming request does not specify a `$select`. This can be desirable when an entity type has many properties or an API producer needs to add properties that are computationally expensive to return by default.

## Solution

For incoming requests targeting an entity type where the caller does not specify a `$select` clause, API producers **may** return a subset of the entity type's properties, omitting computationally expensive properties. To get the non-default properties of an entity type, callers must explicitly request them using `$select`.

The pattern also uses an instance annotation to inform callers that other properties are also available. The same annotation is also used to encourage callers to use `$select`.

## When to use this pattern

API producers should use this pattern when adding expensive or non-performant properties to an existing entity type, or when adding properties to an entity type that has already grown too large (with more than 20 properties).

## Issues and considerations

- Do **not** rely on the `ags:Default` schema annotation for default properties functionality, as this is a legacy implementation. Returning default properties **must** be implemented by API producers.
- Changing a default property to non-default is considered a breaking change.
- One of the challenges with default properties is informing developers that the response does not contain the full set of properties. To solve for this discovery problem, if the response contains default properties only, then:
  - the response **must** contain a `@microsoft.graph.tips` instance annotation.
  - the `@microsoft.graph.tips` instance annotation **must** only be emitted if the client uses "developer mode" via the `Prefer: ms-graph-dev-mode` HTTP request header. It is expected that this header will only be used by client developer and scripting tools like Graph Explorer, the Microsoft Graph Postman collections, and Microsoft Graph PowerShell.
  - the `@microsoft.graph.tips` instance annotation value **must** contain "This request only returns a subset of the resource's properties. Your app will need to use $select to return non-default properties. To find out what other properties are available for this resource see https://learn.microsoft.com/graph/api/resources/{entityTypeName}".
- Callers must be able to use `$filter` with non-default properties, even though they won't show up by default in the response.

Additionally, for incoming requests targeting an entity type where the caller does not specify a `$select` clause, the API Gateway Service will inject a `@microsoft.graph.tips` instance annotation, informing callers to use $select, when in "developer mode".
API producers who use [response passthrough](https://dev.azure.com/msazure/One/_wiki/wikis/Microsoft%20Graph%20Partners/391069/Enabling-response-passthrough) must also implement this behavior, supplying the same information as shown in the [examples section below](#calling-an-api-without-using-select).

## Examples

In this example we'll use the following `channel` entity type.

```xml
<EntityType Name="channel" BaseType="graph.entity">
  <Property Name="createdDateTime" Type="Edm.DateTimeOffset"/>
  <Property Name="description" Type="Edm.String"/>
  <Property Name="displayName" Type="Edm.String" Nullable="false"/>
  <Property Name="email" Type="Edm.String"/>
  <Property Name="isFavoriteByDefault" Type="Edm.Boolean"/>
  <Property Name="membershipType" Type="graph.channelMembershipType"/>
  <!-- moderationSettings is a new computed property that is very expensive -->
  <Property Name="moderationSettings" Type="graph.channelModerationSettings"/> 
  <Property Name="webUrl" Type="Edm.String"/>
  <Property Name="filesFolderWebUrl" Type="Edm.String"/>
</EntityType>
```

In this scenario, the API producer wants to add the `moderationSettings` property to the `channel` entity type.
But when paging through 1000 channels at a time, this additional property will introduce a considerable increase in the response times.
The API producer will use the default properties pattern here, and **not** return `moderationSettings` by default.

### Calling an API with default properties

In this example, the caller, using Graph Explorer, does not use $select, and the API returns just the default properties.

#### Request

```http
GET /teams/{id}/channels
Prefer: ms-graph-dev-mode
```

#### Response

```http
200 ok
Content-type: application/json
```

```json
{
    "@odata.context": "https://graph.microsoft.com/beta/$metadata#Collection(microsoft.graph.channel)",
    "@microsoft.graph.tips": "This request only returns a subset of the resource properties. Your app will need to use $select to return non-default properties. To find out what other properties are supported for this resource, please see the Properties section in https://learn.microsoft.com/graph/api/resources/channel.",
    "value": [
        {
            "displayName": "My First Shared Channel",
            "description": "This is my first shared channels",
            "id": "19:PZC_kAPAm12RPBMkEaJyXaY_d2PE6mJV6MzO1EiCbnk1@thread.tacv2",
            "membershipType": "shared",
            "email": "someemail@dot.com",
            "webUrl": "webUrl-value",
            "filesFolderWebUrl": "sharePointUrl-value",
            "tenantId": "tenantId-value",
            "isFavoriteByDefault": null,
            "createdDateTime": "2019-08-07T19:00:00Z"
        },
        {
            "displayName": "My Second Private Channel",
            "description": "This is my second shared channels",
            "id": "19:PZC_kAPAm12RPBMkEaJyXaY_d2PE6mJV6MzO1EiCbnk2@thread.tacv2",
            "membershipType": "private",
            "email": "someemail2@dot.com",
            "webUrl": "webUrl-value2",
            "filesFolderWebUrl": "sharePointUrl-value2",
            "tenantId": "tenantId-value",
            "isFavoriteByDefault": null,
            "createdDateTime": "2019-08-09T19:00:00Z"
        }
    ]
}
```

In the response, we can see that `moderationSettings` is not being returned. Additionally, the API producer is returning a `tips` instance annotation, informing the caller that this response only returns default properties, how to get the non-default properties, and where to find information about this type's properties. The `tips` instance annotation is only emitted if the `Prefer: ms-graph-dev-mode` HTTP request header is present.

### Calling an API with default properties and $select

In this example, the caller needs `moderationSettings` for their API scenario.  They try this out in Graph Explorer first.

#### Request

```http
GET /teams/{id}/channels?$select=id,membershipType,moderationSettings
Prefer: ms-graph-dev-mode
```

#### Response

```http
200 ok
Content-type: application/json
```

```json
{
    "@odata.context": "https://graph.microsoft.com/beta/$metadata#Collection(microsoft.graph.channel)",
    "value": [
        {
            "id": "19:PZC_kAPAm12RPBMkEaJyXaY_d2PE6mJV6MzO1EiCbnk1@thread.tacv2",
            "membershipType": "shared",
            "channelModerationSettings": {
                "userNewMessageRestriction": "everyone",
                "replyRestriction": "everyone",
                "allowNewMessageFromBots": true,
                "allowNewMessageFromConnectors": true
            }
        },
        {
            "id": "19:PZC_kAPAm12RPBMkEaJyXaY_d2PE6mJV6MzO1EiCbnk2@thread.tacv2",
            "membershipType": "private",
            "channelModerationSettings": {
                "userNewMessageRestriction": "moderators",
                "replyRestriction": "authorAndModerators",
                "allowNewMessageFromBots": true,
                "allowNewMessageFromConnectors": true
            }
        }
    ]
}
```

In this case, because the request has a `$select`, the `tips` instance annotation is not emitted.

### Calling an API without using $select

The caller makes a `GET` request without $select, to an API that doesn't have any default properties, via Graph Explorer.

#### Request

```http
GET /me/todo/lists
Prefer: ms-graph-dev-mode
```

#### Response

```http
200 ok
Content-type: application/json
```

```json
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users('99a6e897-8c54-4354-a739-626fbe28ed78')/todo/lists",
    "@microsoft.graph.tips": "Use $select to choose only the properties your app needs, as this can lead to performance improvements. For example: GET me/todo/lists?$select=displayName,isOwner",
    "value": [
        {
            "@odata.etag": "W/\"c5yMNreru0OMO71/IwuKGQAG6WUnjQ==\"",
            "displayName": "Tasks",
            "isOwner": true,
            "isShared": false,
            "wellknownListName": "defaultList",
            "id": "AAMkADU3NTBhNWUzLWE0MWItNGViYy1hMTA0LTkzNjRlYTA2ZWI2ZAAuAAAAAAAFup0i-hqtR5N14AJlh2qTAQATqGUvrHrTEbWPAKDJQ2mMAAACWIG1AAA="
        },
        {
            "@odata.etag": "W/\"c5yMNreru0OMO71/IwuKGQAG6WUnmQ==\"",
            "displayName": "Outlook Commitments",
            "isOwner": true,
            "isShared": false,
            "wellknownListName": "none",
            "id": "AQMkADU3NTBhNWUzLWE0MWItNGViYy1hMTA0LTkzNjRlYTA2ZWI2ZAAuAAADBbqdIv4arUeTdeACZYdqkwEAc5yMNreru0OMO71-IwuKGQABWbOTpQAAAA=="
        }
    ]
}
```

Notice how for this scenario, where there are no default properties and the caller does not use `$select`, there's a `tips` instance annotation, encouraging the app developer to use `$select`. This `tips` annotation is automatically added to the response by the API gateway service, as long as the workload service doesn't use response passthrough (in which case it is the responsibility of the workload service).
