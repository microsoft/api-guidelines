# Core Types

## Overview

Types exist in Microsoft Graph which are highly-connected/central to the Microsoft Graph ecosystem. Often, these types are in the position of being able to contain structural properties relevant to other APIs, because they are connected to many entities in Microsoft Graph.

Structural properties should be only added to these core types when they are intrinsic to the entity itself, and strictly not for the purpose of convenience due to the entity's position in Microsoft Graph.

## Core Types in Microsoft Graph

The following types are identified as core types, and will require strong justification to allow new structural properties to be added in all cases.

- ```user```
- ```group```
- ```device```

## Alternatives to Adding Structural Properties

Instead of adding a structural property to the existing core type (`user`, `group` or `device`), create a new type that models the information captured in the proposed structural property(s).
Then, model the relationship between the existing core type and the new type by adding a navigation property. For information on modeling with navigation properties, see [Navigation Property](../patterns/navigation-property.md).

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

Model the information by creating a new type and model the relationship to the existing core type with a navigation property. To determine which option is most appropriate, see [Navigation Property](../patterns/navigation-property.md):

#### Option 1: Add a navigation property on the existing core type to the new type, containing the new type.

Define the new entity type:
```xml
<EntityType name="bankAccountDetail">
    <Property Name="accountNumber" Type="Edm.string"/>
    <Property Name="routingNumber" Type="Edm.string"/>
</EntityType>
```

Add a contained navigation from user to the new entity type:
```xml
<EntityType name="user">
    <NavigationProperty Name="bankAccountDetail" Type="bankAccountDetail" ContainsTarget="true"/>
</EntityType>
```

#### Option 2: Contain the new type in an entity set elsewhere, and add a navigation property to the new type on the existing core type.

Define the new entity type:
```xml
<EntityType name="bankAccountDetail">
    <Property Name="accountNumber" Type="Edm.string"/>
    <Property Name="routingNumber" Type="Edm.string"/>
</EntityType>
```

Contain the new entity type in an entity set or singleton:
```xml
<EntitySet Name="bankAccountDetails" EntityType="bankAccountDetail">
```

Add a navigation from user to the new type:
```xml
<EntityType name="user">
    <NavigationProperty Name="bankAccountDetail" Type="bankAccountDetail" />
</EntityType>
```

#### Option 3: Contain the new type in an entity set elsewhere, and add a navigation property to the existing core type on the new type.

Define the new entity type, with a navigation to the user:
```xml
<EntityType name="bankAccountDetail">
    <Property Name="accountNumber" Type="Edm.string"/>
    <Property Name="routingNumber" Type="Edm.string"/>
    <NavigationProperty Name="user" Type="microsoft.graph.user" />
</EntityType>
```

Contain the new entity type in an entity set or singleton:
```xml
<EntitySet Name="bankAccountInformations" EntityType="bankAccountInformation">
```
