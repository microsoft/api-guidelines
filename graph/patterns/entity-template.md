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
<EntityContainer ...>
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
A client can then create a `foo` from a `fooTemplate` by calling:
```http
POST /foos/create
{
  "template@odata.bind": "/templates/{templateId}"
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

If a property `frob` is added to the `foo` entity:
```xml
<EntityType Name="foo">
  ...
  <Property Name="frob" Type="self.frob" Nullable="true" />
</EntityType>
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
