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

A new template can be created by specifying `frob`:
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








then the associated `fooTemplate` type should be updated to accommodate `frob`.
It is tempting to just add `frob` to `fooTemplate`.
There are 4 important cases to consider with the `frob` property:
1. If `frob` is provided as `null`, then `frob` is assigned a default value by the service that is dynamic based on the service state and the customer configuration
2. If `frob` is provided as `null`, then `frob` is assigned the value of `null`
3. If `frob` is not provided, then `frob` is assigned a default value by the service that is dynamic based on the service state and the customer configuration
4. If `frob` is not provided, then `frob` is assigned the value of `null`

Because these cases can be used in combination with each other (for example, 3 and 4 can coexist), templates need to accommodate these situations.
As a result, if 3 and 4 coexist, adding a `frob` property to the `fooTemplate` changes the semantics of existing instances of the `fooTemplate`; any existing template now needs to have the `frob` property backfilled to some concrete default value, or `null`, but `null` is different from the value not being provided.
This means that we cannot just add a `frob` property to the `fooTemplate`.
//// TODO is the conclusion then that we should have "not provided" types for templates?
//// TODO use a not provided instance annotation


### TODO start of scratch pad

```xml
<ComplexType Name="templateProperty" Abstract="true" />
<ComplexType Name="notProvidedTemplateProperty" BaseType="self.templateProperty" />

<ComplexType Name="fizzTemplateProperty" BaseType="self.templateProperty">
  <Property Name="value" Type="self.fizz" />
</ComplexType>
<ComplexType Name="buzzTemplateProperty" BaseType="self.templateProperty">
  <Property Name="value" Type="self.buzz" />
</ComplexType>

      <EntityType Name="fooTemplate">
        <Key>
          <PropertyRef Name="id" />
        </Key>
        <Property Name="id" Type="Edm.String" Nullable="false" />
        <Property Name="fizz" Type="self.fizzTemplateProperty" />
        <Property Name="buzz" Type="self.buzzTemplateProperty" />
      </EntityType>
```


```http
POST /fooTemplates
{
  "fizz": {
    "@odata.type": "#self.fizzTemplateProperty",
	"value": {
	  // fizz properties here
	}
  }
}

HTTP/1.1 201 Created
Location: /fooTemplates/{templateId1}

{
  "id": "{templateId1}",
  "fizz": {
    "@odata.type": "#self.fizzTemplateProperty",
	"value": {
	  // fizz properties here
	}
  },
  "buzz": {
    "@odata.type": "#self.notProvidedTemplateProperty"
  }
}
```

```xml
<Action Name="create" IsBound="true">
  <Parameter Name="bindingParameter" Type="Collection(self.fooTemplate)" Nullable="false" />
  <Parameter Name="foo" Type="self.foo" Nullable="false" />
  <ReturnType Type="self.fooTemplate" />
</Action>
```

```http
POST /fooTemplates/create
{
  "foo": {
    "fizz": {
	  // fizz properties here
	}
  }
}

HTTP/1.1 201 Created
Location: /fooTemplates/{templateId1}

{
  "id": "{templateId}",
  "fizz": {
    "@odata.type": "#self.fizzTemplateProperty",
	"value": {
	  // fizz properties here
	}
  },
  "buzz": {
    "@odata.type": "#self.notProvidedTemplateProperty"
  }
}
```

### TODO end of scratch pad





//// TODO do you want to talk about "non-provided" properties? if `fizz` isn't in the template creation, it becomes `null`; do we need a way to say "don't use `fizz` when creating the instance from the template"?
//// TODO establish a general pattern for actions bound to an entity collection where the actions are different constructor overloads; the "original" overload is still just a post to the collection
//// TODO templates are just a new constructor overload
//// TODO managing templates

## When to use this pattern

*Describe when and why the solution is applicable and when it might not be.*

## Issues and considerations

TODO if managing templates is more consuming that managing the entities

## Example

*Provide a short example from real life.*
