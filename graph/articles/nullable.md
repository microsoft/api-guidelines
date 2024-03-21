# Nullable Properties

A nullable property means *only* that the property may have `null` as a value; the "nullability" of a property does not say anything about how a value is set into a property.
For example, a non-nullable property is *not* required to create a new instance of an entity.
It only means that the property will have a value when it is retrieved.
In the case that no value is provided when the entity is created, this means that the service will create one; this value can be specified with the `DefaultValue` attribute, but if the value is contextual and determine at request time, then the property can both be non-nullable *and* have no `DefaultValue` specified. 
Below are some examples of nullable and non-nullable properties. 

## CSDL

```xml
<EntitySet Name="servicePrincipals" Type="self.servicePrincipal" />
...
<EntityType Name="servicePrincipal">
  <Key>
    <PropertyRef Name="id" />
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false" />
  <Property Name="appId" Type="Edm.String" Nullable="false" /> <!--required for creation-->
  <Property Name="displayName" Type="Edm.String" Nullable="false" />
  <Property Name="foo" Type="Edm.String" Nullable="true" DefaultValue="testval" />
  <Property Name="bar" Type="Edm.String" Nullable="false" DefaultValue="differentvalue" />
  ...
</EntityType>
```

## HTTP Requests

### 1. Create a servicePrincipal with no properties

```HTTP
POST /servicePrincipals

400 Bad Request
{
  "error": {
    "code": "badRequest",
    "message": "The 'appId' property is required to create a servicePrincipal."
  }
}
```

### 2. Create a servicePrincipal without a display name

```HTTP
POST /servicePrincipals
{
  "appId": "00000000-0000-0000-0000-000000000001"
}

201 Created
{
  "appId": "00000000-0000-0000-0000-000000000001",
  "displayName": "some application name",
  "foo": "testval",
  "bar": "differentvalue",
  ...
}
```
Notes:
1. `displayName` was given a value by the service even though no value was provided by the client
2. `foo` has the default value as specified by its `DefaultValue` attribute in the CSDL
3. `bar` has the default value as specified by its `DefaultValue` attribute in the CSDL

### 3. Update the display name of a service principal to null

```HTTP
PATCH /servicePrincipals/00000000-0000-0000-0000-000000000001
{
  "displayName": null
}

400 Bad Request
{
  "error": {
    "code": "badRequest",
    "message": "null is not a valid value for the property 'displayName'; 'displayName' is not a nullable property."
  }
}
```
Notes:
1. `displayName` cannot be set to `null` because it has be marked with `Nullable="false"` in the CSDL.

### 4. Update the display name of a service principal

```HTTP
PATCH /servicePrincipals/00000000-0000-0000-0000-000000000001
{
  "displayName": "a non-generated display name"
}

200 OK
{
  "appId": "00000000-0000-0000-0000-000000000001",
  "displayName": "a non-generated display name",
  "foo": "testval",
  "bar": "differentvalue",
  ...
}
```
Notes:
1. `displayName` can be set to any value other than `null`
2. The response body here is provided for clarity, and is not part of the guidance itself. The [OData v4.01 standard](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html#sec_UpdateanEntity) states that the workload can decide the behavior.

### 5. Update the foo property of a service principal to null

```HTTP
PATCH /servicePrincipals/00000000-0000-0000-0000-000000000001
{
  "foo": null
}

200 OK
{
  "appId": "00000000-0000-0000-0000-000000000001",
  "displayName": "a non-generated display name",
  "foo": null,
  "bar": "differentvalue",
  ...
}
```
Notes:
1. `foo` can be set to `null` because it has be marked with `Nullable="true"` in the CSDL.
2. The response body here is provided for clarity, and is not part of the guidance itself. The [OData v4.01 standard](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html#sec_UpdateanEntity) states that the workload can decide the behavior.

### 6. Update the foo property of a service principal to a non-default value

```HTTP
PATCH /servicePrincipals/00000000-0000-0000-0000-000000000001
{
  "foo": "something other than testval"
}

200 OK
{
  "appId": "00000000-0000-0000-0000-000000000001",
  "displayName": "a non-generated display name",
  "foo": "something other than testval",
  "bar": "differentvalue",
  ...
}
```
Notes:
1. `foo` can be set to `something other than testval`
2. The response body here is provided for clarity, and is not part of the guidance itself. The [OData v4.01 standard](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html#sec_UpdateanEntity) states that the workload can decide the behavior.

### 7. Update the bar property of a service principal to null

```HTTP
PATCH /servicePrincipals/00000000-0000-0000-0000-000000000001
{
  "bar": null
}

400 Bad Request
{
  "error": {
    "code": "badRequest",
    "message": "null is not a valid value for the property 'bar'; 'bar' is not a nullable property."
  }
}
```
Notes:
1. `bar` cannot be set to `null` because it has be marked with `Nullable="false"` in the CSDL.

### 8. Update the bar property of a service principal to a non-default value

```HTTP
PATCH /servicePrincipals/00000000-0000-0000-0000-000000000001
{
  "bar": "a new bar"
}

200 OK
{
  "appId": "00000000-0000-0000-0000-000000000001",
  "displayName": "a non-generated display name",
  "foo": "something other than testval",
  "bar": "a new bar",
  ...
}
```
Notes:
1. `bar` can be set to `a new bar`
2. The response body here is provided for clarity, and is not part of the guidance itself. The [OData v4.01 standard](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html#sec_UpdateanEntity) states that the workload can decide the behavior.

### 9. Create a service principal while customizing the display name
```HTTP
POST /servicePrincipals
{
  "appId": "00000000-0000-0000-0000-000000000001",
  "displayName": "a different name"
}

201 Created
{
  "appId": "00000000-0000-0000-0000-000000000001",
  "displayName": "a different name",
  "foo": "testval",
  "bar": "differentvalue",
  ...
}
```
Notes:
1. `displayName` isn't required to create a new `servicePrincipal`, but it *can* be provided; this is orthogonal to whether or not the property has `Nullable="true"` or `Nullable="false"`.
2. `foo` has the default value as specified by its `DefaultValue` attribute in the CSDL
3. `bar` has the default value as specified by its `DefaultValue` attribute in the CSDL

### 10. Create a service principal with a null display name
```HTTP
POST /servicePrincipals
{
  "appId": "00000000-0000-0000-0000-000000000001",
  "displayName": null
}

400 Bad Request
{
  "error": {
    "code": "badRequest",
    "message": "null is not a valid value for the property 'displayName'; 'displayName' is not a nullable property."
  }
}
```
Notes:
1. `displayName` isn't required to create a new `servicePrincipal`, but it *can* be provided; it *cannot* be provided as `null` because the property was marked with `Nullable="false"`

### 11. Create a service principal with a value for the foo property
```HTTP
POST /servicePrincipals
{
  "appId": "00000000-0000-0000-0000-000000000001",
  "foo": "a foo value on creation"
}

201 Created
{
  "appId": "00000000-0000-0000-0000-000000000001",
  "displayName": "some application name",
  "foo": "a foo value on creation",
  "bar": "differentvalue",
  ...
}
```
Notes:
1. `displayName` was given a value by the service even though no value was provided by the client
2. `foo` isn't required to create a new `servicePrincipal`, but it *can* be provided; this is orthogonal to whether or not the property has `Nullable="true"` or `Nullable="false"`.
3. `bar` has the default value as specified by its `DefaultValue` attribute in the CSDL

### 12. Create a service principal with null for the foo property
```HTTP
POST /servicePrincipals
{
  "appId": "00000000-0000-0000-0000-000000000001",
  "foo": null
}

201 Created
{
  "appId": "00000000-0000-0000-0000-000000000001",
  "displayName": "some application name",
  "foo": null,
  "bar": "differentvalue",
  ...
}
```
Notes:
1. `displayName` was given a value by the service even though no value was provided by the client
2. `foo` isn't required to create a new `servicePrincipal`, but it *can* be provided; because the property has `Nullable="true"`, a `null` value can be provided for it.
3. `bar` has the default value as specified by its `DefaultValue` attribute in the CSDL

### 13. Create a service principal with a value for the bar property
```HTTP
POST /servicePrincipals
{
  "appId": "00000000-0000-0000-0000-000000000001",
  "bar": "running out of ideas for value names"
}

201 Created
{
  "appId": "00000000-0000-0000-0000-000000000001",
  "displayName": "some application name",
  "foo": "testval",
  "bar": "running out of ideas for value names",
  ...
}
```
Notes:
1. `displayName` was given a value by the service even though no value was provided by the client
2. `foo` has the default value as specified by its `DefaultValue` attribute in the CSDL
3. `bar` isn't required to create a new `servicePrincipal`, but it *can* be provided; this is orthogonal to whether or not the property has `Nullable="true"` or `Nullable="false"`.

### 14. Create a service principal with null for the bar property
```HTTP
POST /servicePrincipals
{
  "appId": "00000000-0000-0000-0000-000000000001",
  "bar": null
}

400 Bad Request
{
  "error": {
    "code": "badRequest",
    "message": "null is not a valid value for the property 'bar'; 'bar' is not a nullable property."
  }
}
```
Notes:
1. `bar` isn't required to create a new `servicePrincipal`, but it *can* be provided; it *cannot* be provided as `null` because the property was marked with `Nullable="false"`
