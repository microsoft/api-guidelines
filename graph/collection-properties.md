OData services treat collections of complex types differently than they treat collections of entity types. 
This is due to the nature of entity types being "individually addressable" (they have some key which uniquely identifies them within the collection) while complex types are not individually addressable. 
The result of these differences is that it is almost always preferable to use entity types for collections rather than complex types; please see the exceptions [here](#exceptions).
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
  ]
}
```

## Adding individual elements to a collection

For both `foos` and `bars`, the OData standard specifies that elements can be added to the collection using a `POST` request
1. [Complex Types](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html#sec_UpdateaCollectionProperty):

   > A successful POST request to the edit URL of a collection property adds an item to the collection.
2. [Entity Types](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html#_Toc31358976)

   > To create an entity in a collection, the client sends a POST request to that collection's URL.

### Complex Type

#### Add an element to the collection

```HTTP
POST  /interestingData/foos
{
  "someProperty": "a value"
}

204 No Content
```

#### Check the new contents of the collection

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

#### Add an element to the collection

```HTTP
POST  /interestingData/bars
{
  "differentProperty": 42
}

204 No Content
Location: /interestingData/bars/thirdBarId
```

#### Check the new contents of the collection

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
    },
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

There is no way to do this for complex types because there is no way to address a particular instance of a complex type within a collection. 

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

There is no way to do this for complex types because there is no way to address a particular instance of a complex type within a collection. 

### Entity Type

#### Remove an element from the collection

```HTTP
DELETE  /interestingData/bars/thirdBarId

204 No Content
```

#### Check the new contents of the collection

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
  ]
}
```

## Updating individual elements in a collection

The [OData standard](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html#sec_UpdateanEntity) specifies that clients can update individual elements of a collection of entity types using a `PATCH` request:
> To update an individual entity, the client makes a PATCH or PUT request to a URL that identifies the entity.

There is no way to do this for complex types because there is no way to address a particular instance of a complex type within a collection. 

### Entity Type

#### Update an element in the collection

```HTTP
PATCH  /interestingData/bars/firstBarId
{
  "differentProperty": "15"
}

204 No Content
```

#### Check the new contents of the collection

```HTTP
GET  /interestingData/bars

200 OK
{
  "value": [
    {
      "id": "firstBarId",
      "differentProperty": 15
    },
    {
      "id": "secondBarId",
      "differentProperty": -6914
    }
  ]
}
```

## Updating a collection

The [OData standard](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html#sec_UpdateanEntity) specifies that clients can replace all elements of a collection of complex types or all elements of a collection of entity types using a `PATCH` request:
> Collection properties...provided in the payload corresponding to updatable properties MUST replace the value of the corresponding property in the entity or complex type.

The standard also specifies that collections of entities can be updated in a relative fashion using the [delta syntax](https://docs.oasis-open.org/odata/odata-json-format/v4.01/odata-json-format-v4.01.html#_Toc38457777) in a `PATCH` request:
> The body of a PATCH request to a URL identifying a collection of entities...MUST contain the context control information...and...MUST contain an array-valued property named value containing all added, changed, or deleted entities...

TODO the dstandard also blah blah delta path blah

### Complex Type

#### Replace the elements in a collection

```HTTP
PATCH /interestingData
{
  "foos": [
    {
      "someProperty": "a replacement value"
    },
    {
      "someProperty": "more replacement value"
    },
    {
      "someProperty": "a new value demonstrating that additional elements can be in the collection"
    }
  ]
}

204 No Content
```

#### Check the new contents of the collection

```HTTP
GET /interestingData/foos

200 OK
{
  "value": [
    {
      "someProperty": "a replacement value"
    },
    {
      "someProperty": "more replacement value"
    },
    {
      "someProperty": "a new value demonstrating that additional elements can be in the collection"
    }
  ]
}
```

### Entity Type

#### Replace the elements in a collection

```HTTP
PATCH /interestingData
{
  "bars": [
    {
      "id": "fourthBarId",
      "differentProperty": 20
    }
  ]
}

204 No Content
```

#### Check the new contents of the collection

```HTTP
GET /interestingData/bars

200 OK
{
  "value": [
    {
      "id": "fourthBarId",
      "differentProperty": 20
    },
    {
      "id": "fifthBarId",
      "differentProperty": -99999
    }
  ]
}
```

#### Change the elements in a collection

```HTTP
PATCH /interestingData/bars
{
  "@context": "#$delta",
  "value": [
    {
      "id": "fifthBarId",
      "differentProperty": 1024
    },
    {
      "id": "sixthBarId",
      "differentProperty": 9801
    },
    {
      "id": "fourthBarId",
      "@removed": {}
    }
  ]
}

204 No Content
```

#### Check the new contents of the collection

```HTTP
GET /interestingData/bars

200 OK
{
  "value": [
    {
      "id": "fifthBarId",
      "differentProperty": 1024
    },
    {
      "id": "sixthBarId",
      "differentProperty": 9801
    }
  ]
}
```

## Exceptions

TODO

## Conclusion

Collections of entity types have several behaviors that are not available for collections of complex types:
1. Individual elements can be retrieved
2. Individual elements can be removed
3. Individual elements can be updated
4. Several elements within the collection may be added, removed, or updated in a single request

Further, to remove or update elements in a collection of complex types, the entire collection must be replaced with a `PATCH` request. 
This has the potential to result in data loss for clients who accidentally don't include all of the current elements of the collection, or encounter a race condition where two clients are attempting to `PATCH` the same collection.
Due to the increased flexiblity of collections of entity types, and the data loss risks for collections of complex types, collections of entity types should be used. 
