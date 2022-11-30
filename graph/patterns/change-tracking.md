# Change tracking

Microsoft Graph API Design Pattern

*The change tracking pattern provides the ability for API consumers to request changes in data from Microsoft Graph without having to re-read data that has not changed.*


## Problem

API consumers require an efficient way to acquire changes to data in the Microsoft Graph, for example to synchronize an external store or to drive a change-centric business process.

## Solution

API designers can enable the change tracking (delta) capability on a resource in the Microsoft Graph (typically on an entity collection or a parent resource) by declaring a delta function on that resource and applying `Org.OData.Capabilities.V1.ChangeTracking` annotation.

This function returns a delta payload. A delta payload consists of a collection of annotated full or partial Microsoft Graph entities plus either a `nextLink` to further pages of original or change data that are immediately available OR a `deltaLink` to poll to get the next set of changes as they occur.

Annotations allow the delta payload to indicate resources or links which have been deleted. API callers are expected to differentiate resource adds from updates by interpreting the id property of the change records against the existence of resources in whatever external system is doing the processing.

The pattern requires a sequence of requests on the delta function, for additional details see [Change Tracking](https://learn.microsoft.com/en-us/graph/delta-query-overview?tabs=http#use-delta-query-to-track-changes-in-a-resource-collection):

  1. GET request which returns the first page of the current state of the resources that delta applies to.  
  2. [Optionally] Further GET requests to retrieve more pages of the current state via the `@odata.nextLink` URL.
  3. After some time, a GET request to see if there are new changes via the `@odata.deltaLink`URL.
  4. [Optionally] GET requests to retrieve more pages of changes via the `@odata.nextLink` URL.

The `nextLink` provides a mechanism to do server-driven paging through the change data that is currently available.  When there are no further pages of changes immediately available, a `deltaLink` is returned instead.

The `deltaLink` provides a mechanism for the API consumer to catch up on changes since their last request to the delta function, `deltaLink` or last page’s `nextLink`. If no changes have happened since the last request, then the deltaLink MUST return an empty collection.

Both `nextLink` and `deltaLink` MUST be considered opaque URLs. The best practice is to make them opaque via encoding.

Delta payload requirements
  - The payload is a collection of change records using the collection format.
  - The change records are full or partial representations of the resources according to their resource types.
  - When a change representing a resource update is included in the payload the API producer MAY return either the changed properties or the full entity. The ID of the resource MUST be included in every change record.
  - When an entity is deleted, the delta function MUST return the ID of the deleted entity as well as an `@removed` annotation with the reason field.
  - When an entity is deleted, the reason MUST be set to “changed” if the entity can be restored.
  - When an entity is deleted. the reason MUST be set to “deleted” if the entity cannot be restored.
  - There is no mechanism to indicate that a resource has entered or exited the dataset based on a change that causes it to match or no longer match any `$filter` query parameter.
  - When a link to an entity is deleted, when the linked entity is deleted, or when a link to an entity is added, the implementer MUST return a `property@delta` annotation. 
  - When a link to an entity is deleted, but the entity still exists, the reason MUST be set to `changed`.
  - When a link to an entity is deleted along with the entity, the reason MUST be set to `deleted`.
  
API producers MAY choose to collate multiple changes to the same resource into a single change record. 

## When to use this pattern

API consumers want a pull mechanism to request and process change to Microsoft Graph data, either via proactive polling or by responding to Microsoft Graph notifications.

API consumers need guaranteed data integrity over the set of changes to Microsoft Graph data.

## Issues and considerations

 - API service MAY be able to respond to standard OData query parameters with the initial call to the delta function:

    - `$select` to enforce the set of properties on which change is reported.
    - `$filter` to influence the scope of changes returned.
    - `$expand` to include linked resources with the set of changes.
    - `$top` parameter to influence the size of the set of change records.
  
    These query parameters MUST be encoded into subsequent `@odata.nextLink` or `@odata.deltaLink`, such that the same options are preserved through the call sequence without callers respecifying them, which MUST NOT be allowed. OData query parameters must be honored in full, or a 400-error returned.

- Making a sequence of calls to a delta function followed by the opaque URLs in the `nextLink` and `deltaLink` MUST guarantee that the data at the start time of the call sequence and all changes to the data thereafter will be returned at least once. It is not necessary to avoid duplicates in the sequence. When the delta function is returning changes, they MUST be sequenced chronologically refer to [public documentation](https://learn.microsoft.com/en-us/graph/delta-query-overview?view=graph-rest-1.0) for more details.

- The delta function can be bound to a collection, as with
`/users/delta` that returns the changes to the users' collection.

  or to some logical parent resource, where the change records are implied to be relative to all collections contained within the parent, for example
  `/me/planner/all/delta` – this returns changes to any resource within planner that a user is subscribed to as a heterogenous collection.

- Although this capability is similar to the OData `$delta` feed capability, it is a different construct. Microsoft Graph APIs MUST provide change tracking through the delta function and MUST NOT implement the OData `$delta` feed when providing change tracking capabilities to ensure the uniformity of the API experience.
- API producers might use `$skipToken` and `$deltaToken` within their implementations of `nextLink` and `deltaLink`, however the URLs are defined as being opaque and the existence of the tokens MUST NOT be documented.   It is not a breaking change to modify the structure of `nextLinks` or `deltaLinks`.- 
- `nextLink` and `deltaLink` URLs are valid for a specific period before the client application needs to run a full synchronization again.For `nextLink`, a minimal validity time should be 1 hour. For `deltaLink`, a minimal validity time should be seven days. When a link is no longer valid it must return a standard error with a 410 GONE response code.
  

## Alternatives

- Change notifications pattern with rich payloads – for use cases where API consumers would find calling back into Microsoft Graph onerous and absolute integrity guarantees are less critical.


## Examples

### Change tracking on entity set

```
<Function Name="delta" IsBound="true" ags:OwnerService="Microsoft.DirectoryServices">
        <Parameter Name="bindingParameter" Type="Collection(graph.user)" />
        <ReturnType Type="Collection(graph.user)" />
</Function>
<EntitySet Name="users" EntityType="Microsoft.DirectoryServices.user"> 
    <Annotation Term="Org.OData.Capabilities.V1.ChangeTracking"> 
      <Record> 
        <PropertyValue Property="Supported" Bool="true" /> 
      </Record> 
    </Annotation> 
</EntitySet> 
```

### Change tracking on navigation property

```
<EntityType Name="educationRoot" ags:OwnerService="Microsoft.EducationRosteringAPIs" ags:WorkloadIds="Microsoft.EducationAssignment,Microsoft.EducationDataSync">
    <NavigationProperty Name="classes" Type="Collection(graph.educationClass)" ContainsTarget="true" />
    <NavigationProperty Name="me" Type="graph.educationUser" ContainsTarget="true" />
    <NavigationProperty Name="schools" Type="Collection(graph.educationSchool)" ContainsTarget="true" />
    <NavigationProperty Name="synchronizationProfiles" Type="Collection(graph.educationSynchronizationProfile)" ContainsTarget="true" ags:OwnerService="Microsoft.EducationDataSync" />
    <NavigationProperty Name="users" Type="Collection(graph.educationUser)" ContainsTarget="true" />
</EntityType>
<Function Name="delta" IsBound="true" ags:OwnerService="Microsoft.EducationRosteringAPIs">
    <Parameter Name="bindingParameter" Type="Collection(graph.educationClass)" />
    <ReturnType Type="Collection(graph.educationClass)" />
</Function>
 <Annotations Target="microsoft.graph.educationRoot/classes">
    <Annotation Term="Org.OData.Capabilities.V1.ChangeTracking" ags:OwnerService="Microsoft.EducationRosteringAPIs">
      <Record>
        <PropertyValue Property="Supported" Bool="true" />
      </Record>
    </Annotation>
</Annotations>
```

### Delta payload

 Here after the initial delta call, a user resource is updated, and there is one user added to and one removed from that user’s directReports collection. Additionally, a second user is deleted. In this case, there are no further pages of change records currently available. For detailed sequence of requests see [Change Tracking](https://learn.microsoft.com/en-us/graph/delta-query-overview?tabs=http#use-delta-query-to-track-changes-in-a-resource-collection).

```
GET https://graph.microsoft.com/v1.0/users/delta?$skiptoken=pqwSUjGYvb3jQpbwVAwEL7yuI3dU1LecfkkfLPtnIjvB7XnF_yllFsCrZJ

{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users",
    "@odata.deltaLink": "https://graph.microsoft.com/v1.0/users/delta?$deltatoken=mS5DuRZGjVL-abreviated",
    "value": [
        {
            "businessPhones": ["+1 309 555 0104"],
            "displayName": "Grady Archie", 
            "givenName": "Grady", 
            "jobTitle": "Designer", 
            "mail": "GradyA@contoso.onmicrosoft.com", 
            "officeLocation": "19/2109", 
            "preferredLanguage": "en-US", 
            "surname": "Archie", 
            "userPrincipalName": "GradyA@contoso.onmicrosoft.com", 
            "id": "0baaae0f-b0b3-4645-867d-742d8fb669a2", 
            "directReports@delta": [ 
                { 
                    "@odata.type": "#microsoft.graph.user", 
                    "id": "99789584-a1e1-4232-90e5-866170e3d4e7" 
                } ,
                { 
                    "id": "66789583-f1e2-6232-70e5-366170e3d4a6",
                    "@removed": {
                        "reason": "deleted"
                    }
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