# Dictionary

Microsoft Graph API Design Pattern

*The Dictionary type provides the ability to create a set of primitives or objects, of the same type, where the API consumer can define a name for each value in the set.*

## Problem
--------

The API design requires a resource to include an unknown quantity of data elements of the same type that must be named using values provided by the API consumer.

## Solution
--------

API designers use a JSON object to represent a dictionary in a `application/json`response payload. When describing the model in CSDL, a new complex type can be created that derives from `Org.OData.Core.V1.Dictionary` and then use the `Org.OData.Validation.V1.OpenPropertyTypeConstraint`to constrain the type that can be used for the values in the dictionary.

Dictionary entries can be added via `POST`, updated via `PATCH`, and they can be removed by setting the entry value to `null`. Multiple entries can be updated at once by using `PATCH` on the dictionary property.

## Issues and Considerations
-------------------------

Dictionaries, sometimes called maps, are a collection of name-value pairs. They allow dynamic data sets to be accessed in a systematic manner and are a good compromise between a strictly defined ahead of time structure with all its named properties and between a loosely defined dynamic object (i.e. OData OpenTypes).

As dictionary entries are removed via setting the value to null, this means that dictionaries can only support values that are non-nullable.

  OpenQuestions:
  - Can/should PUT be supported on the dictionary property and/or the entry value 
  - What does OData say about being able to POST to a structured property? Will OData Web API allow that?
  - Must an implementer support PATCH at both the dictionary level and the entry level?

More information:

- [OData reference](https://github.com/oasis-tcs/odata-vocabularies/blob/master/vocabularies/Org.OData.Core.V1.md#dictionary)

 
## When to Use this Pattern
------------------------

Before using a dictionary type in your API definition make sure your scenario fits the following criteria:

- The data values MUST be related to one another semantically as a collection.
- The value types MUST be a primitive type or is a **ComplexType**. Mixed primitive types are not allowed.
- The client MUST define the keys of this type. As opposed to the service defining it in advance.

### Alternatives to consider

- [Open extensions](https://docs.microsoft.com/en-us/graph/extensibility-open-users) when you want to provide clients the ability to extend Microsoft Graph.
- [Open types](https://docs.microsoft.com/en-us/aspnet/web-api/overview/odata-support-in-aspnet-web-api/odata-v4/use-open-types-in-odata-v4) when your data is not a collection in nature.
- [Complex types](https://docs.microsoft.com/en-us/odata/webapi/complextypewithnavigationproperty) when the set of data values are known.

## Examples
-------

### JSON payload example

The following example illustrates the resulting JSON for a property of dictionary type. The parent object has been ommitted for brievety.

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

In this set of examples we're modeling a **roles** property of dictionary type on the user entity which is exposed by the users entity set.

#### Getting an entry from the dictionary

```HTTP
GET https://graph.microsoft.com/v1.0/users/10/roles/author
```

Reponse:

```json
{
  "domain": "contoso"
}
```

#### Getting the dictionary

```HTTP
GET https://graph.microsoft.com/v1.0/users/10/roles
```

Reponse:

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

#### Getting the entity with the dictionary

```HTTP
GET https://graph.microsoft.com/v1.0/users/10
```

Reponse:

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

#### Creating an entry in the dictionary

```HTTP
POST https://graph.microsoft.com/v1.0/users/10/roles/author

{
  "domain": "contoso"
}
```

#### Updating the dictionary

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

> Note: setting one of the keys to **null** deletes it from the dictionary.
> Note: the domain values for the existing author and maintainer entries will get udpated.
> Note: the reviewer entry will be inserted in the dictionary.

#### Updating an entry in the dictionary

```HTTP
PATCH https://graph.microsoft.com/v1.0/users/10/roles/author

{
  "domain": "fabrikam"
}
```

#### Deleting an entry from the dictionary

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
  <Annotation Term="SupportedHttpMethod">
    <Collection><!-- use this annotation to indicate you want the SDKs to generate additional request builders to update the dictionary atomtically -->
      <String>GET</String>
      <String>PATCH</String>
      <String>DELETE</String>
      <String>POST</String>
    <Collection>
  </Annotation>
</ComplexType>
```

### Additional information

[SDK implementation guidance](./client-guidance.md).
