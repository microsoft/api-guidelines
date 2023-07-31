# Snapshots

Microsoft Graph API Design Pattern

*The snapshots pattern allows access to data for an individual resource even when that resource no longer exists, while still allowing to easily correlate references to that resource.*

## Problem

There are often times when data on an entity (usually a subset) needs to be exposed even after the entity has been removed or updated. 
This data is representating that entity during a snapshot in time. 
However, simply duplicating the data prevents correlating that snapshot data with the entity itself or with other data that are also snapshots in time.

An example would be a log of `user`s who have signed in.
The log could have an entry for when the `user` signed in, as well as details about the user.
But without a way to correlate this sign in data to the `user` itself, further investigations becomes difficult.

## Solution

The solution is to have a complex type which represents the resource as a snapshot in time, while also providing a navigation property to the resource itself.
The following CSDL demonstrates this with the `user` sign in example:

```xml
<EntityType Name="user">
  <Key>
    <PropertyRef Name="id" />
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false" />
  <Property Name="displayName" Type="Edm.String" Nullable="false" />
  <Property Name="userPrincipalName" Type="Edm.String" Nullable="false" />
  <!--
  Other existing propeties go here.
  -->
</EntityType>

<EntityType Name="userSignIn">
  <Key>
    <PropertyRef Name="id" />
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false" />
  <Property Name="attemptDateTime" Type="Edm.DateTimeOffset" Nullable="false" />
  <Property Name="requestedPermissions" Type="Collection(Edm.String)" Nullable="false" />
  <Property Name="user" Type="self.userSnapshot" Nullable="false" />
</EntityType>

<ComplexType Name="userSnapshot">
  <Property Name="id" Type="Edm.String" Nullable="false" />
  <Property Name="displayName" Type="Edm.String" Nullable="false" />
  <Property Name="userPrincipalName" Type="Edm.String" Nullable="false" />
  <!--
  For this example, only these properties are being stored in the snapshot. Any subset of properties from `user` is valid to store in a snapshot type.
  -->
  <NavigationProperty Name="user" Type="self.user" Nullable="true" /> <!--This is nullable because the user may no longer exist.-->
</ComplexType>
```

## When to use this pattern

//// TODO give alternative, anti-pattern approaches
*Describe when and why the solution is applicable and when it might not be.*

## Issues and considerations

*Describe tradeoffs of the solution.*

## Example

*Provide a short example from real life.*
