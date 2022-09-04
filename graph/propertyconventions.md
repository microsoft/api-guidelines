# Guidelines for the conventions of properties on graph types

## Updating read-only properties

### Convention

It is a common call pattern for Microsoft Graph clients to `GET` an entity, make changes to a few properties, and then `PATCH` the same entity with the updated values.
When clients do this, they often ignore that some properties are read-only or that a property may have a value that is allowed in a response, but not in a request.
Because of this, workloads should support the practice that a request that updates a property should be considered a no-op if the value in the request matches the current value of the property.

### Example

For the following schema:
```xml
<EntityType Name="entity">
  <Key>
    <PropertyRef Name="id" />
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false" />
  <Property Name="foo" Type="Edm.String" Nullable="false" /> <!--this property is read-only-->
  <Property Name="bar" Type="Edm.String" Nullable="false" />
</EntityType>
```
A client retrieves an `entity`:
```http
GET /entities/{id}

HTTP/1.1 200 OK
{
  "id": "...",
  "foo": "somevalue",
  "bar": "someothervalue"
}
```
If the client attempts to update `foo`, they will receive an error:
```http
PATCH /entities/{id}
{
  "foo": "somenewvalue"
}

HTTP/1.1 400 Bad Request
{
    "error": {
        "code": "ReadOnlyProperty",
        "message": "The property 'foo' is read-only and cannot be updated."
    }
}
```
However, if the client were to update `bar` while including `foo` but leaving it the same value, the request will be successful:
```http
PATCH /entities/{id}
{
  "foo": "somevalue",
  "bar": "somenewvalue"
}

HTTP/1.1 204 No Content
```
A subsequent `GET` call would reflect the change in `bar`:
```http
GET /entities/{id}

HTTP/1.1 200 OK
{
  "id": "...",
  "foo": "somevalue",
  "bar": "somenewvalue"
}
```