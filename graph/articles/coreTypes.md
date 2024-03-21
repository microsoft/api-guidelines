# Core Types

## Overview

Types exist in graph which are highly-connected/central to the graph ecosystem. Often, these types are the position of containing many structural properties relevant to other APIs, because they are connected to many entities in the graph.

Structural properties should be only added to these core types when they are properties of the entity itself and strictly not for the purpose of convenience due to the entity's position in the graph.

## Core Types in Graph

The following types are identified as core types, and will require strong justification to allow new structural properties to be added in all cases.

- ```user```
- ```group```
- ```device```

## Alternatives to Adding Structural Properties

Instead of adding a structural property to an existing type, do the following:
1. Create a new type that models the information captured in the proposed structural property.
2. Create a navigation property linking the new type to the existing type, doing one of the following:
   - Contain the new type in an entity set elsewhere, and add a navigation property to the existing type on the new type.
   - Contain the new type in an entity set elsewhere, and add a navigation property to the new type on the existing type.
   - Add a navigation property to the new type on the existing type, containing the new type.

## Example:

Modeling adding "bank account information", which includes two properties `accountNumber` and `routingNumber`, to entity type ```user```.

### Don't:

Don't add new properties to core types such as `user`.

```xml
<EntityType name="user">
    <Property Name="accountNumber" Type="Edm.string"/>
    <Property Name="routingNumber" Type="Edm.string"/>
</EntityType>
```

### Do:

Instead, create a new entity modeling the concept that you wish to introduce, and create a navigation property connecting that entity to the desired type.

```xml
<EntityType name="bankAccountInformation">
    <Property Name="accountNumber" Type="Edm.string"/>
    <Property Name="routingNumber" Type="Edm.string"/>
</EntityType>

...

<EntityType name="user">
    <NavigationProperty Name="bankAccountInformation" Type="bankAccountInformation" ContainsTarget="true"/>
</EntityType>
```
