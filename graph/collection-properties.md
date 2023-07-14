OData services treat collections of complex types differently than they treat collections of entity types. 
This is due to the nature of entity types being "individually addressable" (they have some key which uniquely identifies them within the collection) while complex types are not individually addressable. 
Let's use the following CSDL as an example:

```xml
<EntityType Name="interestingData">
  <Property Name="foos" Type="Collection(self.foo)" Nullable="false" />
  <NavitationProperty Name="bars" Type="Collection(self.bar)" Nullable="false" ContainsTarget="true" />
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

## Adding elements to a collection

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

200 OK
{
  "value": [
    {
      "someProperty": "a value"
    }
  ]
}
```

### Entity Type

```HTTP
POST  /interestingData/bars
{
  "differentProperty": 42
}

200 OK
{
  "value": [
    {
      "id": "firstBarId",
      "differentProperty": 42
    }
  ]
}
```
