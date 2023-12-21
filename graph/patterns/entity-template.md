# Entity template

Microsoft Graph API Design Pattern

*An entity template is a template which can be used to create a new instance of an entity.*

## Problem

A customer wants to create instances of an entity that all start from the same basic data, and this data needs to be shared across clients over time.

## Solution

### Introduce a new entity type that is a template for the entity that the customer is trying to create. 
For an existing `foo` entity:
```xml
<EntityContainer Name="Container">
  <EntitySet Name="foos" EntityType="self.foo" />
</EntityContainer>
<EntityType Name="foo">
  <Key>
    <PropertyRef Name="id" />
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false" />
  <Property Name="fizz" Type="self.fizz" />
  <Property Name="buzz" Type="self.buzz" />
</EntityType>
```

a "template" entity is defined for `foo`:
```xml
<EntityType Name="fooTemplate">
  <Key>
    <PropertyRef Name="id" />
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false" />
  <Property Name="fizz" Type="self.fizz" />
  <Property Name="buzz" Type="self.buzz" />
</EntityType>
<EntityContainer Name="Container">
  <EntitySet Name="fooTemplates" EntityType="self.fooTemplate" />
</EntityContainer>
```

An action should also be introduced that will create the `foo` based on the `fooTemplate`:
```xml
<Action Name="create" IsBound="true">
  <Parameter Name="bindingParameter" Type="Collection(self.foo)" Nullable="false" />
  <Parameter Name="template" Type="self.fooTemplate" Nullable="false" />
  <ReturnType Type="self.foo" />
</Action>
```

A client can now create a `fooTemplate`, something like:
```http
POST /fooTemplates
{
  "fizz": {
    // fizz properties here
  },
  "buzz": null
}

HTTP/1.1 201 Created
Location: /fooTemplates/{templateId1}

{
  "id": "{templateId1}",
  "fizz": {
    // fizz properties here
  },
  "buzz": null
}
```

This template can then be used to create a `foo`:
```http
POST /foos/create
{
  "template@odata.bind": "/fooTemplates/{templateId1}"
}

HTTP/1.1 201 Created
Location: /foos/{fooId}

{
  "id": "{fooId}",
  "fizz": {
    ...
  },
  "buzz": {
    ...
  }
}
```

### Add a new property to an existing entity and update its associated template entity

A property `frob` may be added to the `foo` entity:
```xml
<EntityType Name="foo">
  ...
  <Property Name="frob" Type="self.frob" Nullable="true" />
</EntityType>
```

Likely, clients will want an analogous property on the `fooTemplate`.
Because the default value of such a property on `foo` is ambiguous (the default may be `null`, some static value, or a service-generated value only known based on the overall state at runtime), the `fooTemplate` needs a clear way to indicate whether a value was specified for the `frob` property.
This is done with the use of the `notProvided` instance annotation. //// TODO link to docs about instance annotations, and figure out the correct name for "notProvided"
`frob` will be defined on `fooTemplate` as usual:
```xml
<EntityType Name="fooTemplate">
  ...
  <Property Name="frob" Type="self.frob" Nullable="true" />
</EntityType>
```

Now, existing templates will return the `notProvided` instance annotation for `frob`:
```http
GET /fooTemplates/{templateId1}

HTTP/1.1 200 OK
{
  "id": "{templateId1}",
  "fizz": {
    // fizz properties here
  },
  "buzz": null,
  "frob@notProvided": true
}
```

A new template can be created in the following ways

#### a value for `frob` is specified by the client
```
POST /fooTemplates
{
  "fizz": {
    // fizz properties here
  },
  "buzz": {
    // buzz properties here
  },
  "frob": {
    // frob properties here
  }
}

HTTP/1.1 201 Created
Location: /fooTemplates/{templateId2}

{
  "id": "{templateId2}",
  "fizz": {
    // fizz properties here
  },
  "buzz": {
    // buzz properties here
  },
  "frob": {
    // frob properties here
  }
}
```

//// TODO show a foo created from this template

### `null` is explicitly specified for `frob`

//// TODO

### no value is specified for `frob`

//// TODO




//// TODO make sure you know how to implement this in webapi











//// TODO establish a general pattern for actions bound to an entity collection where the actions are different constructor overloads; the "original" overload is still just a post to the collection
//// TODO templates are just a new constructor overload

## When to use this pattern

*Describe when and why the solution is applicable and when it might not be.*

## Issues and considerations

TODO if managing templates is more consuming that managing the entities

## Example

*Provide a short example from real life.*
