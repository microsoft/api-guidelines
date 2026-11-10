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
<!--because actions cannot be overloaded, and for a consistent client experience, the name "createFromTemplate" should be used instead of other names-->
<Action Name="createFromTemplate" IsBound="true">
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
POST /foos/createFromTemplate
{
  "template": {
    "@id": "/fooTemplates/{templateId1}"
  }
}

HTTP/1.1 201 Created
Location: /foos/{fooId1}

{
  "id": "{fooId1}",
  "fizz": {
    ...
  },
  "buzz": {
    ...
  }
}
```

You can find an example of how to implement this using OData WebApi [here](https://github.com/OData/AspNetCoreOData/commit/d9fa5db0177e1ee8e71a87641ad097dae3ee5e5d).

### Add a new property to an existing entity and update its associated template entity

A property `frob` may be added to the `foo` entity:
```xml
<EntityType Name="foo">
  ...
  <Property Name="frob" Type="self.frob" Nullable="true" /> <!--NOTE: this property is nullable for illustrative purposes; new properties do not need to be marked nullable-->
</EntityType>
```

Likely, clients will want an analogous property on the `fooTemplate`.
Because the default value of such a property on `foo` is ambiguous (the default may be `null`, some static value, or a service-generated value only known based on the overall state at runtime), the `fooTemplate` needs a clear way to indicate whether a value was specified for the `frob` property.
This is done with the use of the `notProvided` instance annotation.
`frob` will be defined on `fooTemplate` as usual:
```xml
<EntityType Name="fooTemplate">
  ...
  <Property Name="frob" Type="self.frob" Nullable="true" /> <!--NOTE: this property is nullable for illustrative purposes; new properties do not need to be marked nullable-->
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

The client creates a new template with a value for `frob`:
```http
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

The client then uses that template to create a `foo`:
```http
POST /foos/createFromTemplate
{
  "template@odata.bind": "/fooTemplates/{templateId2}"
}

HTTP/1.1 201 Created
Location: /foos/{fooId2}

{
  "id": "{fooId2}",
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

#### `null` is explicitly specified for `frob`

The client creates a new template specifying `null` for `frob`:
```http
POST /fooTemplates
{
  "fizz": {
    // fizz properties here
  },
  "buzz": {
    // buzz properties here
  },
  "frob": null
}

HTTP/1.1 201 Created
Location: /fooTemplates/{templateId3}

{
  "id": "{templateId3}",
  "fizz": {
    // fizz properties here
  },
  "buzz": {
    // buzz properties here
  },
  "frob": null
}
```

The client then uses that template to create a `foo`:
```http
POST /foos/createFromTemplate
{
  "template@odata.bind": "/fooTemplates/{templateId3}"
}

HTTP/1.1 201 Created
Location: /foos/{fooId3}

{
  "id": "{fooId3}",
  "fizz": {
    // fizz properties here
  },
  "buzz": {
    // buzz properties here
  },
  "frob": null
}
```

#### no value is specified for `frob`

The client creates a new template without specifying any value for `frob`:
```http
POST /fooTemplates
{
  "fizz": {
    // fizz properties here
  },
  "buzz": {
    // buzz properties here
  }
}

HTTP/1.1 201 Created
Location: /fooTemplates/{templateId4}

{
  "id": "{templateId4}",
  "fizz": {
    // fizz properties here
  },
  "buzz": {
    // buzz properties here
  },
  "frob@notProvided": true
}
```

The client then uses that template to create a `foo`:
```http
POST /foos/createFromTemplate
{
  "template@odata.bind": "/fooTemplates/{templateId4}"
}

HTTP/1.1 201 Created
Location: /foos/{fooId4}

{
  "id": "{fooId3}",
  "fizz": {
    // fizz properties here
  },
  "buzz": {
    // buzz properties here
  },
  "frob": // server-generated default value for frob here
}
```

You can find an example of how to implement this using OData WebApi [here](https://github.com/OData/AspNetCoreOData/commit/cf5583a5fd2c8acedf178df90d6dd6cab9820e62).

## When to use this pattern

This pattern should be used whenever customers need to create instances of an entity using the same basic outline, but where those instances have their own, individual lifecycle.

## Issues and considerations

Templates for relatively small entities should be avoided.
Generally speaking, managing templates should not be more costly for clients than directly managing the entities themselves.
