# Viewpoint

Microsoft Graph API Design Pattern


*The viewpoint pattern provides the ability to manage properties of a shared object that have different values for different users.*

## Problem
A shared resource, such as a website or a group message, may have different states for different users who access it at different times in an organizational context. For example, user1 may read and delete a message, while user2 may not have seen it yet. This usually happens when a shared item is presented in an individual context.
## Solution

The viewpoint pattern provides a solution to how to model an individual user context on a shared resource using a `viewpoint` structural property on an API entity type.
For example, the `viewpoint` property can indicate whether a message is read, deleted, or flagged for a given user. 
The consistent naming convention ensures that when a developer uses Graph APIs all `viewpoint` structural properties represent type specific user context across different M365 services and features.

This pattern simplifies the API client logic by hiding the state transition details and providing state persistency on the server side. The server can manage the different viewpoints for the shared resource without exposing additional complexity to the client. To support queries for a user state the `viewpoint` property should support filtering.
## Issues and considerations

- Because the `viewpoint` property reflects an individual user's context, it is null when accessed with application permissions.
- Sometimes, the viewpoint can be computed on the server. In this case, an API producer should add OData annotations to the property to provide more information for downstream tools, such as SDKs and documentation generation.
```
    <Annotations Target="microsoft.graph.approvalItem/viewPoint">
        <Annotation Term="Org.OData.Core.V1.Computed" Bool="true" />
    </Annotations>
```
- An alternative to this design would be to store the user state on the client side. However, this may be problematic in some cases, because of the many devices that a user may have and the need to synchronize the state across them.
- Often, updating the `viewpoint` property may cause a side effect, so you might consider an OData action to do the update. For some user scenarios, the `PATCH` method could be a better way to update a `viewpoint`.

## Examples

### Defining a viewpoint

The following example demonstrates how to define the 'viewpoint' property for the `chat` entity, where a chat is a collection of chatMessages between one or more participants: 
```
  <ComplexType Name="chatViewpoint" >
        <Property Name="isHidden" Type="Edm.Boolean" />
        <Property Name="lastMessageReadDateTime" Type="Edm.DateTimeOffset" />
  </ComplexType>

  <EntityType Name="chat" BaseType="graph.entity" >
        <Property Name="chatType" Type="graph.chatType" Nullable="false" />
        <Property Name="createdDateTime" Type="Edm.DateTimeOffset" />
        <Property Name="lastUpdatedDateTime" Type="Edm.DateTimeOffset" />              
        <Property Name="topic" Type="Edm.String" />
        <Property Name="viewpoint" Type="graph.chatViewpoint" />
       ...
        <NavigationProperty Name="tabs" Type="Collection(graph.teamsTab)" ContainsTarget="true" />
  </EntityType>

```
### Reading an entity with a viewpoint

The following example shows reading a collection of chats for an identified user, with a viewpoint for each chat:

```http
GET https://graph.microsoft.com/v1.0/users/8b081ef6-4792-4def-b2c9-c363a1bf41d5/chats
```

```http

HTTP/1.1 200 OK
Content-type: application/json
```

```
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#chats",
    "@odata.count": 3,
    "value": [
        {
            "id": "19:meeting_MjdhNjM4YzUtYzExZi00OTFkLTkzZTAtNTVlNmZmMDhkNGU2@thread.v2",
            "topic": "Meeting chat sample",
            "createdDateTime": "2020-12-08T23:53:05.801Z",
            "lastUpdatedDateTime": "2022-12-08T23:58:32.511Z",
            "chatType": "meeting",         
            "viewpoint":{
                "lastMessageReadDateTime": "2021-03-28T21:10:00.000Z"              
            }
        },
        {
            "id": "19:561082c0f3f847a58069deb8eb300807@thread.v2",
            "topic": "Group chat sample",
            "createdDateTime": "2020-12-03T19:41:07.054Z",
            "lastUpdatedDateTime": "2020-12-08T23:53:11.012Z",
            "chatType": "group",            
            "viewpoint":{
                "lastMessageReadDateTime": "0000-01-01T00:00:00.000Z"                
            }
        }
    ]
}
```
### Updating a viewpoint using an action

The following example shows marking a chat `viewpoint` as read for a user using an action:

```http

POST https://graph.microsoft.com/beta/chats/19:7d898072-792c-4006-bb10-5ca9f2590649_8ea0e38b-efb3-4757-924a-5f94061cf8c2@unq.gbl.spaces/markChatReadForUser

{
 "user": {
    "id" : "d864e79f-a516-4d0f-9fee-0eeb4d61fdc2",
    "tenantId": "2a690434-97d9-4eed-83a6-f5f13600199a"
  }
}
```

The server responds with a  success status code and no payload:

```http
HTTP/1.1 204 No Content
```
### Updating a viewpoint using `PATCH` method

The following example shows how to mark a topic with the `viewpoint` label as reviewed for a user by using the `PATCH` method (this example does not represent an actual API, but only an illustration):

```http
PATCH https://graph.microsoft.com/beta/sampleTopics/19:7d898072-792c-4006-bb10-5ca9f259

{  
    "title": "Announcements: Changes to PowerPoint and Word to open files faster",
    ...
    "viewpoint": {
         "isReviewed" : "true"
    }
}
```

The server responds with a  success status code and no payload:

```http
HTTP/1.1 204 No Content
```
