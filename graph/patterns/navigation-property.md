# Navigation Property

Microsoft Graph API Design Pattern

*A navigation property is used to identify a relationship between resources.*

## Problem
--------

It is often valuable to represent a relationship between resources in an API. This may be a many-to-one or a one-to-many relationship. 

Relationships between resources are often implicitly represented by a property contained in one of the resources that identifies the other related resource. Usually that information returned in a representation as an id value and the property is named using a convention that identifies the target type of related resource. e.g. userId 

In many-to-one relationships, for a client to access the related resource it must request the primary resource, read the id value of the related resource and then construct a URL to the related resource using the Id value and knowledge of how the property name maps to the related resource. This requires at least two round trips and requires the client know how to construct the URL to the related resource.

For both many-to-one and one-to-many relationships in order to retrieve information from both resources on both sides of the relationship requires at minimum two requests.

Requiring two round trips to access this information is inefficient for some applications and the lack of a formally described relationship limits the ability for tooling to take advantage of the relationship to improve developer experience.

## Solution
--------

Navigation properties are an [OData convention](https://docs.microsoft.com/en-us/odata/webapi/model-builder-untyped#navigation-property) that allows an API designer to describe a special kind of property in a model that references an related entity. In the HTTP API this property name translates to a path segment that can be appended to the URL of the primary resource in order to access a representation of the related resource. This prevents the client from needing to know any additional information on how to construct the URL to the related resource and the client does not need to retrieve the primary resource if it is only interested in the related resource.  It is the responsibility of the API implementation to determine the Id of the related resource and return the representation of the related entity.

e.g. /user/{userId}/manager  # many-to-one relationship
     /user/{userId}/messages # one-to-many relationship

Additionally, using the OData Expand query parameter, related entities can be transcluded into the primary entity so both can be retrieved in a single round trip.

## Issues and Considerations
-------------------------

In the current Microsoft Graph implementation, support for navigation properties is limited to entities within the same backend service or the user entity.  
 
Navigation properties defined within an entity are not returned when retreiving the representation of an entity.  

Implementing support for accessing the "$ref" of a navigation property allows a caller to return just the URL of related resource. e.g. `/user/23/manager/$ref`. This is useful when a client wishes to identity the related resource but doesn't need all of its properties.

## When to Use this Pattern
------------------------

### "has a" relationships

The use of navigation properties is preferred over including an Id field to reference the related entity in a many-to-one relationship.  Id values require a client to make two round trips to retrieve the details of a related entity.  With a navigation property a client can retrieve a related entity in a single round trip. 

Navigation properties are also useful when clients sometimes want to retrieve both the primary entity and the related entity in a single round trip.  The `expand` query parameter makes this possible.

### "parent-child" relationships

Resources that contain a parent Id property in a child resource can utilize a navigation property in the parent resource that is declared as a collection of child resources. If desirable, a parent navigation property can also be created in the child resource to the parent resource. This is usually not necessary as the parent URL is a subset of child resource URL. The main use of this would be when retrieving child resources and choosing to expand properties of the parent resource so that both can be retrieved in a single request.  

`/invoice/{invoiceId}/items/{itemId}?expand=parentInvoice(select=invoiceDate,Customer)`

One other use case is when child resources appear in a non-contained collection and there is a desire to access the canonical parent:

`/me/pinnedChannels/{channelId}/team`

## Example
-------

### Retrieving a related entity

```http
GET /users/{id}/manager

200 OK
Content-Type: application/json

{
  "@odata.type": "#microsoft.graph.user",
  "id": "6b3ee805-c449-46a8-aac8-8ff9cff5d213",
  "displayName": "Bob Boyce"
}
```

### Retrieving a reference to a related entity

```http
GET /users/{id}/manager/$ref

200 OK
Content-Type: application/json

{
  "@odata.id": "https://graph.microsoft.com/v1.0/directoryObjects/6b3ee805-c449-46a8-aac8-8ff9cff5d213/Microsoft.DirectoryServices.User"
}
```
Note: Currently the base URL returned in $ref results are incorrect.  In order to process these URLs the client will need to convert the URL to a Graph URL.

### Retrieving an entity with a related entity included

```http
GET /users/{id}?select=id,displayName&expand=manager(select=id,displayName)

200 OK
Content-Type: application/json

{
  "id": "3f057904-f936-4bf0-9fcc-c1e6f84289d8",
  "displayName": "Jim James",
  "manager": {
    "@odata.type": "#microsoft.graph.user",
    "id": "6b3ee805-c449-46a8-aac8-8ff9cff5d213",
    "displayName": "Bob Boyce"
  }
}
```

### Creating an entity with a reference to a related entity

Create a new user that references an existing manager
```http
POST /users/{id}
Content-Type: application/json

{
    "displayName": "Bob",
    "manager@bind": "https://graph.microsoft.com/v1.0/users/{managerId}"
}

201 Created
```

Create a new user and the users manager and create a relationship between the two.

```http
POST /users/{id}
Content-Type: application/json

{
    "displayName": "Jim James",
    "manager": {
        "displayName": "Bob Boyce"
    }
}

201 Created
```

### Updating a related entity reference

Update the user entity to contain a relationship to an existing manager.
 
```http
PATCH /users/{id}
Content-Type: application/json

{
    "displayName": "Bob",
    "manager@bind": "https://graph.microsoft.com/v1.0/users/{managerId}"
}

204 No Content
```

Create a relationship between the user and the existing manager.

```http
PUT /users/{id}/manager/$ref
Content-Type: application/json

{
    "@OData.Id": "https://graph.microsoft.com/v1.0/users/{managerId}"
}

204 No Content
```
 
### Clear a related entity reference

Remove the relationship between the user and the manager.
 
```http
DELETE /users/{id}/manager/$ref

204 No Content
```

Delete the related entity.

```http
DELETE /users/{id}/manager

204 No Content
```
