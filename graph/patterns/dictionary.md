# Dictionary

Microsoft Graph API Design Pattern

*The dictionary type provides the ability to create a set of primitives or objects of the same type where the API consumer can define a name for each value in the set.*

## Problem

The API design requires a resource to include an unknown quantity of data elements of the same type that must be named by using values provided by the API consumer.

## Solution

API designers use a JSON object to represent a dictionary in an `application/json` response payload. When describing the model in CSDL, a new complex type can be created that derives from `Org.OData.Core.V1.Dictionary` and then uses the `Org.OData.Validation.V1.OpenPropertyTypeConstraint` to constrain the type that can be used for the values in the dictionary.

Dictionary entries can be added via `POST`, updated via `PATCH`, and removed by setting the entry value to `null`. Multiple entries can be updated at the same time by using `PATCH` on the dictionary property.

## When to use this pattern

Before using a dictionary type in your API definition, make sure that your scenario fits the following criteria:

- The data values MUST be related to one another semantically as a collection.
- The value types MUST be a primitive type or a **ComplexType**. Mixed primitive types are not allowed.
- The client MUST define the keys of this type, as opposed to the service defining them in advance.

### Alternatives

- [Open extensions](https://docs.microsoft.com/graph/extensibility-open-users) when you want to provide clients the ability to extend Microsoft Graph.
- [Complex types](https://docs.microsoft.com/odata/webapi/complextypewithnavigationproperty) when the set of data values are known.

## Issues and considerations

Dictionaries, sometimes called maps, are a collection of name-value pairs. They allow dynamic data sets to be accessed in a systematic manner and are a good compromise between a strictly defined-ahead-of-time structure with all its named properties and a loosely defined dynamic object (such as OData OpenTypes).

Because dictionary entries are removed by setting the value to `null`, dictionaries can only support values that are non-nullable.

Open questions:

- Can/should PUT be supported on the dictionary property and/or the entry value?
- What does OData say about being able to POST to a structured property? Will OData Web API allow that?
- Must an implementer support PATCH at both the dictionary level and the entry level?

For more information, see the [OData reference](https://github.com/oasis-tcs/odata-vocabularies/blob/master/vocabularies/Org.OData.Core.V1.md#dictionary).

## Examples

### JSON payload example

The following example illustrates the resulting JSON for a property of dictionary type. The parent object has been omitted for brevity.

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

### HTTP calls examples

In this set of examples, we model a **roles** property of dictionary type on the user entity, which is exposed by the users entity set.

#### Get an entry from the dictionary

```HTTP
GET https://graph.microsoft.com/v1.0/users/10/roles/author
```

Response:

```json
{
  "domain": "contoso"
}
```

#### Get the dictionary

```HTTP
GET https://graph.microsoft.com/v1.0/users/10/roles
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

#### Get the entity with the dictionary

```HTTP
GET https://graph.microsoft.com/v1.0/users/10
```

Response:

```json
{
  "id": "10",
  "displayName": "Jane Smith",
  "roles": {
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

#### Create an entry in the dictionary

```HTTP
POST https://graph.microsoft.com/v1.0/users/10/roles/author

{
  "domain": "contoso"
}
```

#### Update the dictionary

```HTTP
PATCH https://graph.microsoft.com/v1.0/users/10/roles

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

> **Notes:**
>
> - Setting one of the keys to **null** deletes it from the dictionary.
> - The domain values for the existing author and maintainer entries are updated.
> - The reviewer entry is inserted in the dictionary.

#### Update an entry in the dictionary

```HTTP
PATCH https://graph.microsoft.com/v1.0/users/10/roles/author

{
  "domain": "fabrikam"
}
```

#### Delete an entry from the dictionary

```HTTP
DELETE https://graph.microsoft.com/v1.0/users/10/roles/author
```

### CDSL example

The following example defines a complex type **roleSettings** as well as a dictionary of which the key will be a string and the value a **roleSettings**.

```xml
<ComplexType Name="roleSettings">
  <Property Name ="domain" Type="Edm.String" Nullable="false" />
</ComplexType>

<ComplexType Name="assignedRoleGroupDictionary" BaseType="Org.OData.Core.V1.Dictionary">
  <!-- Note: Strongly-typed dictionary
  of roleSettings
  keyed by name of roleGroup. -->
  <Annotation Term="Org.OData.Validation.V1.OpenPropertyTypeConstraint">
    <Collection>
      <String>microsoft.graph.roleSettings</String>
    </Collection>
  </Annotation>
</ComplexType>
```

## See also

- [SDK implementation guidance](./dictionary-client-guidance.md)
