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









`foo` is then updated to have a navigation property to a `fooTemplate`:
```
<EntityType Name="foo">
  <Key>
    <PropertyRef Name="id" />
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false" />
  <Property Name="fizz" Type="self.fizz" />
  <Property Name="buzz" Type="self.buzz" />
  <NavigationProperty Name="template" Type="self.fooTemplate" /> //// TODO create-only
</EntityType>
```
In order to create a `foo` from the `fooTemplate` with ID `{templateId}`, the client can call:
```
POST .../foos //// TODO full URL
{
  "template@odata.bind": ".../fooTemplates/{templateId}" //// TODO full URL
//// TODO this has the same problem as the "draft" antipattern where foos now can't be properly validated
}
```




//// TODO establish a general pattern for actions bound to an entity collection where the actions are different constructor overloads; the "original" overload is still just a post to the collection
//// TODO templates are just a new constructor overload
//// TODO managing templates
//// TODO do you want to talk about "non-provided" properties? if `fizz` isn't in the template creation, it becomes `null`; do we need a way to say "don't use `fizz` when creating the instance from the template"?

## When to use this pattern

*Describe when and why the solution is applicable and when it might not be.*

## Issues and considerations

TODO if managing templates is more consuming that managing the entities

## Example

*Provide a short example from real life.*
