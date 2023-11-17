# Entity template

Microsoft Graph API Design Pattern

*An entity template is a template which can be used to create a new instance of an entity.*


## Problem

A customer wants to create instances of an entity that all start from the same basic data, and this data needs to be shared across clients over time.

## Solution

For an existing `foo` entity:
```
<EntityType Name="foo">
  <Key>
    <PropertyRef Name="id" />
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false" />
  <Property Name="fizz" Type="self.fizz" />
  <Property Name="buzz" Type="self.buzz" />
</EntityType>
//// TODO entity set
```
a "template" entity is defined for `foo`:
```
<EntityType Name="fooTemplate">
  <Key>
    <PropertyRef Name="id" />
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false" />
  <Property Name="fizz" Type="self.fizz" />
  <Property Name="buzz" Type="self.buzz" />
</EntityType>
//// TODO entity set
```
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

//// TODO managing templates
//// TODO do you want to talk about "non-provided" properties? if `fizz` isn't in the template creation, it becomes `null`; do we need a way to say "don't use `fizz` when creating the instance from the template"?

## When to use this pattern

*Describe when and why the solution is applicable and when it might not be.*

## Issues and considerations

TODO if managing templates is more consuming that managing the entities

## Example

*Provide a short example from real life.*
