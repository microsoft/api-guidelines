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

Instead of adding a structural property to the existing type (`user`, `group` or `device`), create a new type that models the information captured in the proposed structural property(s).
Then, do one of the following:
- Add a navigation property on the existing type to the new type, containing the new type.
- Contain the new type in an entity set elsewhere, and add a navigation property to the new type on the existing type.
- Contain the new type in an entity set elsewhere, and add a navigation property to the existing type on the new type.

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

Do one of the following:

#### Add a navigation property on the existing type to the new type, containing the new type.

Define the new entity type:
```xml
<EntityType name="bankAccountInformation">
    <Property Name="accountNumber" Type="Edm.string"/>
    <Property Name="routingNumber" Type="Edm.string"/>
</EntityType>
```

Add a contained navigation from user to the new entity type:
```xml
<EntityType name="user">
    <NavigationProperty Name="bankAccountInformation" Type="bankAccountInformation" ContainsTarget="true"/>
</EntityType>
```

#### Contain the new type in an entity set elsewhere, and add a navigation property to the new type on the existing type.

Define the new entity type:
```xml
<EntityType name="bankAccountInformation">
    <Property Name="accountNumber" Type="Edm.string"/>
    <Property Name="routingNumber" Type="Edm.string"/>
</EntityType>
```

Contain the new entity type in an entity set or singleton:
```xml
<EntitySet Name="bankAccountInformations" EntityType="bankAccountInformation">
```

Add a navigation from user to the new type:
```xml
<EntityType name="user">
    <NavigationProperty Name="bankAccountInformation" Type="bankAccountInformation" />
</EntityType>
```

#### Contain the new type in an entity set elsewhere, and add a navigation property to the existing type on the new type.

Define the new entity type, with a navigation to the user:
```xml
<EntityType name="bankAccountInformation">
    <Property Name="accountNumber" Type="Edm.string"/>
    <Property Name="routingNumber" Type="Edm.string"/>
    <NavigationProperty Name="user" Type="microsoft.graph.user" />
</EntityType>
```

Contain the new entity type in an entity set or singleton:
```xml
<EntitySet Name="bankAccountInformations" EntityType="bankAccountInformation">
```
