# Dictionary

Microsoft Graph API Design Pattern

_The dictionary type provides the ability to create a set key/value pairs where the set of keys is dynamically specified by the API consumer._

## Problem

The API design requires a resource to include an unknown quantity of data values whose keys are defined by the API consumer.

## Solution

API designers use a JSON object to represent a dictionary in an `application/json` response payload. When describing the model in CSDL, a new complex type can be created that derives from `graph.Dictionary` and optionally uses the `Org.OData.Validation.V1.OpenPropertyTypeConstraint` to constrain the type that can be used for the values in the dictionary as appropriate.

Dictionary entries can be added, removed, or modified via `PATCH` to the dictionary property. Entries are removed by setting the property to `null`.

## When to use this pattern

Before using a dictionary type in your API definition, make sure that your scenario fits the following criteria:

- The data values MUST be related to one another semantically as a collection.
- The values MUST be primitive or complex types.
- The client MUST define the keys of this type, as opposed to the service defining them in advance.

### Alternatives

- [Open extensions](https://docs.microsoft.com/graph/extensibility-open-users) when you want to provide clients the ability to extend Microsoft Graph.
- [Complex types](https://docs.microsoft.com/odata/webapi/complextypewithnavigationproperty) when the set of data values are known.

## Issues and considerations

Dictionaries, sometimes called maps, are a collection of name-value pairs. They allow dynamic data sets to be accessed in a systematic manner and are a good compromise between a strictly defined-ahead-of-time structure with all its named properties and a loosely defined dynamic object (such as OData OpenTypes).

Because dictionary entries are removed by setting the value to `null`, dictionaries don't support null values.

For more information, see the [OData reference](https://github.com/oasis-tcs/odata-vocabularies/blob/master/vocabularies/Org.OData.Core.V1.md#dictionary).

## Examples

### String dictionary

#### CSDL declaration
The following example demonstrates defining a dictionary that can contain string values.

```xml
<Schema Namespace="microsoft.graph"> <!--NOTE: the namespace that declares the Dictionary complex type *must* be microsoft.graph-->
  <ComplexType Name="Dictionary" OpenType="true">
    <Annotation Term="Core.Description" String="A dictionary of name-value pairs. Names must be valid property names, values may be restricted to a list of types via an annotation with term `Validation.OpenPropertyTypeConstraint`." />
  </ComplexType>
</Schema>
<Schema Namespace="WorkloadNamespace">
  <ComplexType Name="stringDictionary" OpenType="true" BaseType="microsoft.graph.Dictionary">
    <Annotation Term="Org.OData.Validation.V1.OpenPropertyTypeConstraint">
      <Collection>
        <String>Edm.String</String>
      </Collection>
    </Annotation>
  </ComplexType>
</Schema>
```

Please note that schema validation will fail due to the casing of `Dictionary`.
This warning should be suppressed.

#### Defining a dictionary property
The following example shows defining a dictionary property, "userTags", on the item entity type.

```xml
<EntityType Name="item">
  ...
  <Property Name="userTags" Type="WorkloadNamespace.stringDictionary"/>
</EntityType>
```

#### Reading a dictionary
Dictionaries are represented in JSON payloads as a JSON object, where the property names are comprised of the keys and their values are the corresponding key values. 

The following example shows reading an item with a dictionary property named "userTags":

```HTTP
GET /item
```
Response:
```json
{
  ...
  "userTags":
  {
    "anniversary": "2002-05-19",
    "favoriteMovie": "Princess Bride"
  }
}
```

#### Setting a dictionary value
The following example shows setting a dictionary value.  If "hairColor" already exists, it is updated, otherwise it is added.

```http
PATCH /item/userTags
```
```json
{
   "hairColor": "purple"
}
```

#### Deleting a dictionary value
A dictionary value can be removed by setting the value to null.
```http
PATCH /item/userTags
```
```json
{
   "hairColor": null
}
```

### Complex typed dictionary

#### CSDL declaration
Dictionaries can also contain complex types whose values may be constrained to a particular set of complex types.

The following example defines a complex type **roleSettings**, an **assignedRoleGroupDictionary** that contains **roleSettings**, and an **assignedRoles** property that uses the dictionary..

```xml
<Schema Namespace="microsoft.graph"> <!--NOTE: the namespace that declares the Dictionary complex type *must* be microsoft.graph-->
  <ComplexType Name="Dictionary" OpenType="true">
    <Annotation Term="Core.Description" String="A dictionary of name-value pairs. Names must be valid property names, values may be restricted to a list of types via an annotation with term `Validation.OpenPropertyTypeConstraint`." />
  </ComplexType>
</Schema>
<Schema Namespace="WorkloadNamespace">
  <EntityType Name="principal">
    ...
    <Property Name="assignedRoles" Type="WorkloadNamespace.assignedRoleGroupDictionary">
  </EntityType>

  <ComplexType Name="roleSettings">
    <Property Name ="domain" Type="Edm.String" Nullable="false" />
  </ComplexType>

  <ComplexType Name="assignedRoleGroupDictionary" BaseType="microsoft.graph.Dictionary">
    <!-- Note: Strongly-typed dictionary of roleSettings keyed by name of roleGroup. -->
    of roleSettings
    keyed by name of roleGroup. -->
    <Annotation Term="Org.OData.Validation.V1.OpenPropertyTypeConstraint">
      <Collection>
        <String>microsoft.graph.roleSettings</String>
      </Collection>
    </Annotation>
  </ComplexType>
</Schema>
```

#### Reading a entity with a complex-typed dictionary

The following example illustrates reading an entity containing the complex-typed dictionary "assignedRoles".

```HTTP
GET /users/10
```

Response:

```json
{
  "id": "10",
  "displayName": "Jane Smith",
  "assignedRoles": {
    "author": {
      "domain": "contoso"
    },
    "maintainer": {
      "domain": "fabrikam"
    },
    "architect": {
      "domain": "adventureWorks"
    }
  }
}
```

#### Reading the dictionary property
The following example shows getting just the "assignedRoles" dictionary property.

```HTTP
GET /users/10/assignedRoles
```

Response:

```json
{
  "author": {
    "domain": "contoso"
  },
  "maintainer": {
    "domain": "fabrikam"
  },
  "architect": {
    "domain": "adventureWorks"
  }
}
```

#### Reading an individual entry from the dictionary
The following example shows reading a single complex-typed entry named "author" from the "assignedRoles" dictionary.

```HTTP
GET /users/10/assingedRoles/author
```

Response:

```json
{
  "domain": "contoso"
}
```

#### Setting an individual entry in the dictionary
The following examples shows updating the dictionary to set the value for the "author" entry. If the "author" entry does not exists it is added with the specified values; otherwise, if the "author" entry already exists, it is updated with the specified values (unspecified values are left unchanged).

```HTTP
PATCH /users/10/assignedRoles/author
```
```json
{
  "author" : {
     "domain": "contoso"
  }
}
```

#### Deleting an individual entry from the dictionary
The following example shows deleting the "author" entry by setting its value to null.

```HTTP
PATCH /users/10/assignedRoles
```
```json
{
  "author": null
}
```

#### Setting multiple dictionary entries
The following example sets values for the "author", "maintainer" and "viewer" entries, and removes the "architect" entry by setting it to null.

```HTTP
PATCH /users/10/assignedRoles
```
```json
{
  "author": {
    "domain": "contoso1"
  },
  "maintainer": {
    "domain": "fabrikam1"
  },
  "reviewer": {
    "domain": "fabrikam"
  },
  "architect": null
}

```

## See also

- [SDK implementation guidance](./dictionary-client-guidance.md)
