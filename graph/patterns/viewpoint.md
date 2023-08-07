# Pattern name

Microsoft Graph API Design Pattern

*Provide a short description of the pattern.*
The viewpoint pattern provides the ability to manage an individual status of a shared object for multiple independent actors.

## Problem

Often in an organizational context a group of people receives a common messages but different users act on this messages at a different point of time.So that if user1 read and deleted the message, for user2 this is still unread. usually it happens when a shared item is presented in an individual context.

## Solution

*Describe how to implement the solution to solve the problem.*

Represents user view points data for a serviceUpdateMessage.
Represents user viewpoints data of the service message. This data includes message status such as whether the user has archived, read, or marked the message as favorite. This property is null when accessed with application permissions.
https://learn.microsoft.com/en-us/graph/api/resources/serviceupdatemessage?view=graph-rest-1.0

## When to use this pattern

*Describe when and why the solution is applicable and when it might not be.*
  <Annotations Target="microsoft.graph.approvalItem/viewPoint">
        <Annotation Term="Org.OData.Core.V1.Computed" Bool="true" ags:OwnerService="Microsoft.Approvals" />
      </Annotations>

## Issues and considerations

*Describe tradeoffs of the solution.*

## Example
https://microsoftgraph.visualstudio.com/onboarding/_search?action=contents&text=chatViewpoint&type=code&lp=code-Project&filters=ProjectFilters%7Bonboarding%7DRepositoryFilters%7Bonboarding%7D&pageSize=25&result=DefaultCollection/onboarding/onboarding/GBmaster//reviews/15279-ga-chat-lastMessagePreview/api.md

  <Annotations Target="microsoft.graph.approvalItem/viewPoint">
        <Annotation Term="Org.OData.Core.V1.Computed" Bool="true" ags:OwnerService="Microsoft.Approvals" />
      </Annotations>
*Provide a short example from real life.*
```
  <ComplexType Name="chatViewpoint" ags:WorkloadIds="Microsoft.Teams.GraphSvc">
        <Property Name="isHidden" Type="Edm.Boolean" />
        <Property Name="lastMessageReadDateTime" Type="Edm.DateTimeOffset" />
  </ComplexType>

  <EntityType Name="chat" BaseType="graph.entity" ags:OwnerService="Microsoft.Teams.GraphSvc">
        <Property Name="chatType" Type="graph.chatType" Nullable="false" />
        <Property Name="createdDateTime" Type="Edm.DateTimeOffset" />
        <Property Name="lastUpdatedDateTime" Type="Edm.DateTimeOffset" />
        <Property Name="onlineMeetingInfo" Type="graph.teamworkOnlineMeetingInfo" />
        <Property Name="tenantId" Type="Edm.String" />
        <Property Name="topic" Type="Edm.String" />
        <Property Name="viewpoint" Type="graph.chatViewpoint" />
        <Property Name="webUrl" Type="Edm.String" />
        <NavigationProperty Name="assignedSensitivityLabel" Type="graph.sensitivityLabel" ContainsTarget="true" ags:IsHidden="true" />
        <NavigationProperty Name="installedApps" Type="Collection(graph.teamsAppInstallation)" ContainsTarget="true" />
        <NavigationProperty Name="lastMessagePreview" Type="graph.chatMessageInfo" ContainsTarget="true" />
        <NavigationProperty Name="members" Type="Collection(graph.conversationMember)" ContainsTarget="true" />
        <NavigationProperty Name="messages" Type="Collection(graph.chatMessage)" ContainsTarget="true" />
        <NavigationProperty Name="operations" Type="Collection(graph.teamsAsyncOperation)" ContainsTarget="true" />
        <NavigationProperty Name="permissionGrants" Type="Collection(graph.resourceSpecificPermissionGrant)" ContainsTarget="true" />
        <NavigationProperty Name="pinnedMessages" Type="Collection(graph.pinnedChatMessageInfo)" ContainsTarget="true" />
        <NavigationProperty Name="tabs" Type="Collection(graph.teamsTab)" ContainsTarget="true" />
  </EntityType>

```



```http
GET https://graph.microsoft.com/v1.0/users/8b081ef6-4792-4def-b2c9-c363a1bf41d5/chats
```

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#chats",
    "@odata.count": 3,
    "value": [
        {
            "id": "19:meeting_MjdhNjM4YzUtYzExZi00OTFkLTkzZTAtNTVlNmZmMDhkNGU2@thread.v2",
            "topic": "Meeting chat sample",
            "createdDateTime": "2020-12-08T23:53:05.801Z",
            "lastUpdatedDateTime": "2020-12-08T23:58:32.511Z",
            "chatType": "meeting",
            "lastMessagePreview": {...}
            ,
            "viewpoint":{
                "lastMessageReadDateTime": "2021-03-28T21:10:00.000Z" // User has unread messages
            }
        },
        {
            "id": "19:561082c0f3f847a58069deb8eb300807@thread.v2",
            "topic": "Group chat sample",
            "createdDateTime": "2020-12-03T19:41:07.054Z",
            "lastUpdatedDateTime": "2020-12-08T23:53:11.012Z",
            "chatType": "group",
            "lastMessagePreview": null, // No message was sent in this group chat yet
            "viewpoint":{
                "lastMessageReadDateTime": "0000-01-01T00:00:00.000Z" // User hasnt read anything since no message was posted
            }
        }
    ]
}
```