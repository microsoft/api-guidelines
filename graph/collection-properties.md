OData services treat collections of complex types differently than they treat collections of entity types. 
This is due to the nature of entity types being "individually addressable" (they have some key which uniquely identifies them within the collection) while complex types are not individually addressable. 
The result of these differences is that it is almost always preferable to use entity types for collections rather than complex types; please see the exceptions [here](TODO).
Let's use the following CSDL as an example:

```xml
<EntityType Name="interestingData">
  <Property Name="foos" Type="Collection(self.foo)" Nullable="false" />
  <NavitationProperty Name="bars" Type="Collection(self.bar)" Nullable="false" ContainsTarget="true" />
  <!--TODO also add an example with primitives-->
</EntityType>

<ComplexType Name="foo">
  <Property Name="someProperty" Type="Edm.String" Nullable="false" />
</ComplexType>

<EntityType Name="bar">
  <Key>
    <PropertyRef Name="id" />
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false" />
  <Property Name="differentProperty" Type="Edm.Int32" Nullable="false" />
</EntityType>
```

## Adding individual elements to a collection

For both `foos` and `bars`, the OData standard specifies that elements can be added to the collection using a `POST` request
1. [Complex Types](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html#sec_UpdateaCollectionProperty):

   > A successful POST request to the edit URL of a collection property adds an item to the collection.
2. [Entity Types](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html#_Toc31358976)

   > To create an entity in a collection, the client sends a POST request to that collection's URL.

### Complex Type

```HTTP
POST  /interestingData/foos
{
  "someProperty": "a value"
}

204 No Content
```

### Entity Type

```HTTP
POST  /interestingData/bars
{
  "differentProperty": 42
}

204 No Content
Location: /interestingData/bars/thirdBarId
```

## Retrieving the elements in a collection

The [OData](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html#_Toc31358935) [standard](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html#_Toc31358947) specifies `foos` and `bars` both can be retrieved using a `GET` request:
> OData services support requests for data via HTTP GET requests.
> 
> ...
> 
> OData services support querying collections of entities, complex type instances, and primitive values.

### Complex Type

```HTTP
GET  /interestingData/foos

200 OK
{
  "value": [
    {
      "someProperty": "an original value"
    },
    {
      "someProperty": "a value"
    }
  ]
}
```

### Entity Type

```HTTP
GET  /interestingData/bars

200 OK
{
  "value": [
    {
      "id": "firstBarId",
      "differentProperty": 10
    },
    {
      "id": "secondBarId",
      "differentProperty": -6914
    }
    {
      "id": "thirdBarId",
      "differentProperty": 42
    }
  ]
}
```

## Retrieving individual elements from a collection

The [OData standard](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html#sec_RequestingIndividualEntities) specifies that clients can retrieve individual elements of a collection of entity types using a `GET` request:
> To retrieve an individual entity, the client makes a GET request to a URL that identifies the entity, e.g. its read URL.

There is no way to do this for complex types because there is no way to identity a particular instance of a complex type within a collection. 

### Entity Type

```HTTP
GET  /interestingData/bars/firstBarId

200 OK
{
  "id": "firstBarId",
  "differentProperty": 42
}
```

## Removing individual elements from a collection

The [OData standard](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html#sec_DeleteanEntity) specifies that clients can remove individual elements of a collection of entity types using a `DELETE` request:
> To delete an individual entity, the client makes a DELETE request to a URL that identifies the entity.

There is no way to do this for complex types because there is no way to identity a particular instance of a complex type within a collection. 

### Entity Type

```HTTP
DELETE  /interestingData/bars/firstBarId

204 No Content
```

## Updating individual elements in a collection

TODO

## Updating a collection

TODO do PATCH for complex type and POST + PATCH for entity types

## Exceptions

TODO

## Conclusion

Collections of entity types have several behaviors that are not available for collections of complex types:
1. Individual elements can be retrieved
2. Individual elements can be removed
3. Individual elements can be updated
4. Several elements within the collection may be added, removed, or updated in a single request

Further, to remove or update elements in a collection of complex types, the entire collection must be replace with a `PATCH` request. 
This has the potential to result in data loss for clients who accidentally don't include all of the current elements of the collection, or encounter a race condition where two clients are attempting to `PATCH` the same collection.
Due to the increased flexiblity of collections of entity types, and the data loss risks for collections of complex types, collections of entity types should be used. 
