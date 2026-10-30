# Navigation Property

Microsoft Graph API Design Pattern

*A navigation property is used to identify a relationship between resources.*

## Problem
--------

It is often valuable to represent a relationship between resources in an API. 

Relationships between resources are often implicitly represented by a property contained in one of the resources that provides a key to a related resource. Usually that information is returned in a representation as an id value and the property is named using a convention that identifies the target type of related resource. e.g. userId 

The use of foreign key properties to describe related resources is a weakly typed mechanism and requires additional information for a developer to traverse the relationship. Discovery of related resources is not trivial.

## Solution
--------

Navigation properties are an [OData convention](https://docs.microsoft.com/en-us/odata/webapi/model-builder-untyped#navigation-property) defined in the [CSDL Specification](https://docs.oasis-open.org/odata/odata-csdl-xml/v4.01/odata-csdl-xml-v4.01.html#_Toc38530365) that allows an API designer to describe a special kind of property in a model that references an related entity. In the HTTP API this property name translates to a path segment that can be appended to the URL of the primary resource in order to access a representation of the related resource. This prevents the client from needing to know any additional information on how to construct the URL to the related resource and the client does not need to retrieve the primary resource if it is only interested in the related resource.  It is the responsibility of the API implementation to determine the Id of the related resource and return the representation of the related entity. For example:

     -  /user/{userId}/manager  represents many-to-one relationship
     -  /user/{userId}/messages represents one-to-many relationship

Additionally, using the OData Expand query parameter, related entities can be nested into the primary entity so both can be retrieved in a single round trip.

These relationships can be described in CSDL as follows:

```xml
<EntityType Name="user">
  <NavigationProperty Name="manager" Type="user" ContainsTarget="false" >
  <NavigationProperty Name="messages" Type="user" ContainsTarget="true" >
</EntityType>
```

## Issues and Considerations
-------------------------

In the current Microsoft Graph implementation, there are scenarios which use navigation properties that cross backend services that have automatic support; there are also some limitations for other scenarios. These limitations are being eliminated over time, but it will be necessary to ensure support for any particular scenario.  [Automatic support and limitations of the current implementation](https://dev.azure.com/msazure/One/_wiki/wikis/Microsoft%20Graph%20Partners/354352/Cross-workload-navigations?anchor=supported-scenarios) are documented internally.
 
Navigation properties defined within an entity are not returned by default when retreiving the representation of an entity unless explicity desired by a service.  The API can consumer can use the `expand` query parameterm, where supported, to retreive both the source and the target entity of the relationship in a single request.

Implementing support for accessing the "$ref" of a navigation property allows a caller to return just the URL of related resource. e.g. `/user/23/manager/$ref`. This is useful when a client wishes to identify the related resource but doesn't need all of its properties.

The strongly-typed nature of navigation properties is valuable for backend services and for client applications, when compared with the weakly-typed foreign key property.
Strong typing allows some documentation and visualizations to be automatically generated, it allows SDK generation, and it allows some automated client code generation; it also prevents the need to store duplicate data on the service side and as a result has improved data consistency across APIs since the duplicate data does not need to be regularly refreshed. 

## When to Use this Pattern
------------------------

### "Many-to-one" relationships  

The use of navigation properties is preferred over including an Id field to reference the related entity in a many-to-one relationship.  Id values require a client to make two round trips to retrieve the details of a related entity.  With a navigation property a client can retrieve a related entity in a single round trip. 
 
Many-to-one relationships are always non-contained relationships as the lifetime of the target cannot depend on the source.

```xml
<EntityType Name="order">
  <NavigationProperty Name="customer" Type="customer" ContainsTarget="false" >
</EntityType>
```


### "Zero-or-one-to-one" relationships  

These navigation properties can be used as a structural organization mechanism to separate properties of an entity in a way that is similar to how complex types are often used. The primary difference being that the target of the navigation property are not returned by default when the source entity is retreived.  The use of the navigation properties over complex properties is preferred when the source and target information comes from different backend APIs.

These relationships must be contained.  

```xml
<EntityType Name="invoice">
  <NavigationProperty Name="paymentDetails" Type="paymentInfo" ContainsTarget="true" >
</EntityType>
```

### "One-to-many" relationships

Resources that contain a parent Id property in a child resource can utilize a navigation property in the parent resource that is declared as a collection of child resources. If desirable, a parent navigation property can also be created in the child resource to the parent resource. This is usually not necessary as the parent URL is a subset of child resource URL. The main use of this would be when retrieving child resources and choosing to expand properties of the parent resource so that both can be retrieved in a single request.  

`/invoice/{invoiceId}/items/{itemId}?expand=parentInvoice(select=invoiceDate,Customer)`

```xml
<EntityType Name="invoice">
  <NavigationProperty Name="items" Type="invoiceItem" ContainsTarget="true" >
</EntityType>

<EntityType Name="invoiceItem">
  <NavigationProperty Name="parentInvoice" Type="invoice" ContainsTarget="false" >
</EntityType>
```

One-to-many relationships may be contained or non-contained relations.


## Example
-------

### Retrieving a related entity

```http
GET /users/{id}/manager?$select=id,displayName

200 OK
Content-Type: application/json

{
  "id": "6b3ee805-c449-46a8-aac8-8ff9cff5d213",
  "displayName": "Bob Boyce"
}
```

This navigation property could be described with the following CSDL: 
```xml
<EntityType Name="user">
  <NavigationProperty Name="manager" Type="graph.user" ContainsTarget="false" >
</EntityType>
```
`ContainsTarget` is set to false for clarity, this is the default value when the attribute is omitted.
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
POST /users
Content-Type: application/json

{
    "displayName": "Bob",
    "manager@odata.bind": "https://graph.microsoft.com/v1.0/users/{managerId}"
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
    "manager@odata.bind": "https://graph.microsoft.com/v1.0/users/{managerId}"
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
