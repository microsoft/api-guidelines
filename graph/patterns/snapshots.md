# Snapshots

Microsoft Graph API Design Pattern

*The snapshots pattern allows access to data for an individual resource even when that resource no longer exists, while still allowing to easily correlate references to that resource.*

## Problem

There are often times when data on an entity (usually a subset) needs to be exposed even after the entity has been removed or updated. 
This data is representating that entity during a snapshot in time. 
However, simply duplicating the data prevents correlating that snapshot data with the entity itself or with other data that are also snapshots in time.

An example would be a log of `user`s who have signed in.
The log could have an entry for when the `user` signed in, as well as details about the user.
But without a way to correlate this signin data to the `user` itself, further investigations becomes difficult.

## Solution

The solution is to have a complex type which represents the resource as a snapshot in time, while also providing a navigation property to the resource itself.
The following CSDL demonstrates this with the `user` signin example:

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
  <Property Name="userSnapshot" Type="self.userSnapshot" Nullable="false" />
</EntityType>

<ComplexType Name="userSnapshot">
  <Property Name="id" Type="Edm.String" Nullable="false" />
  <Property Name="displayName" Type="Edm.String" Nullable="false" />
  <Property Name="userPrincipalName" Type="Edm.String" Nullable="false" />
  <!--
  For this example, only these properties are being stored in the snapshot. Any subset of properties from `user` is valid to store in a snapshot type. The properties should have the same names, types, nullability, etc. as the entity that is referred to.
  -->
  <NavigationProperty Name="user" Type="self.user" Nullable="true" /> <!--This is nullable because the user may no longer exist.-->
</ComplexType>
```

TODO example with nested snapshots?

## When to use this pattern

This pattern should be used whenever one resource is referring to another resource, but the data available for the second resource is specific to a point in time that the first resource is referring to.
Logs, reports, and events are the most common examples of this. 

## Issues and considerations

When these snapshot situations arise, it is common to attempt putting the `user` data on the `userSignIn` directly.
This should be avoided because it gives the impression that the data belongs to the signin event.
It also can cause naming conflicts.
Using our above `user` example, let's say that the `userSignIn` wants to include the `displayName` of the `user`:

```xml
<!--NOTE: this is an anti-pattern example-->
<EntityType Name="user">
  <Key>
    <PropertyRef Name="id" />
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false" />
  <Property Name="displayName" Type="Edm.String" Nullable="false" />
  ...
  <Property Name="createdDateTime" Type="Edm.DateTimeOffset" Nullable="false" />
</EntityType>

<EntityType Name="userSignIn">
  <Key>
    <PropertyRef Name="id" />
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false" />
  <Property Name="attemptDateTime" Type="Edm.DateTimeOffset" Nullable="false" />
  <Property Name="displayName" Type="Edm.String" Nullable="false" />
</EntityType>
```

This CSDL ships to customers who begin using it.
They relate back to the workload that they would find it very useful to have the time the `user` was created in the signin logs so that they can do certain kinds of investigations.
Following the same pattern, we sould update the CSDL:

```xml
<!--NOTE: this is an anti-pattern example-->
<EntityType Name="user">
  <Key>
    <PropertyRef Name="id" />
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false" />
  <Property Name="displayName" Type="Edm.String" Nullable="false" />
  ...
  <Property Name="createdDateTime" Type="Edm.DateTimeOffset" Nullable="false" />
</EntityType>

<EntityType Name="userSignIn">
  <Key>
    <PropertyRef Name="id" />
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false" />
  <Property Name="attemptDateTime" Type="Edm.DateTimeOffset" Nullable="false" />
  <Property Name="displayName" Type="Edm.String" Nullable="false" />
  <Property Name="createdDateTime" Type="Edm.DateTimeOffset" Nullable="false" />
</EntityType>
```

Except now, the `createdDateTime` property *appears* to be indicating when the `userSignIn` was created.
This could have been mitigated if the original CSDL had named the property `userDisplayName`, so the new property would be `userCreatedDateTime`.
However, doing this indicates a clear structuring of the data, which is already known to be the case because the `user` entity exists in the first place.
The guidance would generally be to have a navigation property to the entity referred to in such cases, but more generally the guidance is to factor these properties into their own type.
In the snapshot case, we want to do both: we factor the properties in a "snapshot" complex type, *and* that complex type has a navigation property to the entity being referred to.

## Example

### {1} Retrieve signin events that contain user data

```HTTP
GET /userSignIns

200 OK
{
  "value": [
    {
      "id": "{signinid}",
      "attemptDateTime": "2023-07-31 7:56:00 AM",
      "requestedPermissions": [
        "User.Read"
      ],
      "userSnapshot": {
        "id": "00000000-0000-0000-0000-000000000001",
        "displayName": "some display name",
        "userPrincipalName": "user@domain.com"
      }
    },
    ...
  ]
}
```

### {2} Retrieve the user associated with a signin event

```HTTP
GET /userSignIns/{signinid}/userSnapshot/user

200 OK
{
  "id": "00000000-0000-0000-0000-000000000001",
  "displayName": "some display name",
  "userPrincipalName": "user@domain.com",
  ...
}
```
