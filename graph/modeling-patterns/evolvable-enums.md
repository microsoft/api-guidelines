---
title: Evolvable enums
owners: sanonsen, mastaffo
---

# Adding Members to Enumerations

Microsoft Graph services sometimes want to add a member to an enumeration type. However, there are barriers. First and foremost, some deserializers (including Json.NET) fail if an enumeration property has a value not found in that property's enumeration type. Second, a client may not deal with enumeration values unknown to it. The `Evolvable Enumerations` pattern and implementation allows a member to be safely be added to an enumeration.

## Evolvable Enumerations

An evolvable enumeration contains the sentinel member `unknownFutureValue` after which new enumeration members are added. Consider the following enumeration:

```xml
<EnumType Name="weekday">
  <EnumMember Name="monday"/>
  <EnumMember Name="tuesday"/>
  ...
  <EnumMember Name="sunday"/>
  <EnumMember Name="unknownFutureValue"/>
</EnumType>
```

From this the C# SDK generates:

```csharp
public enum weekday
{
  monday,
  tuesday,
  ...
  sunday,
  unknownFutureValue
}
```

The new enumeration member `newday` is added after `unknownFutureValue`:

```xml
<EnumType Name="weekday">
  <EnumMember Name="monday"/>
  <EnumMember Name="tuesday"/>
  ...
  <EnumMember Name="sunday"/>
  <EnumMember Name="unknownFutureValue"/>
  <EnumMember Name="newday"/>     <!-- new value -->
</EnumType>
```

From this the C# SDK generates:

```csharp
public enum weekday
{
  monday,
  tuesday,
  ...
  sunday,
  unknownFutureValue,
  newday      // new value
}
```

## Methods and client opt-in

On POST, if any enumeration property of the entity contains `unknownFutureValue`, the request will fail with `400 Bad Request`. On PATCH, any enumeration property with value `unknownFutureValue` is ignored--that property is not updated.

Callers signal their ability to process added members by including the `include-unknown-enum-members` preference:

```http
GET /me/calendar
Prefer: include-unknown-enum-members
```

Upon GET, when this header is absent, `unknownFutureValue` is returned to the caller for enumeration property values that are one of the added enumeration members. When this header is present, the enumeration value is returned unchanged.

Upon a filtered GET, when this header is absent, if `unknownFutureValue` appears in a `$filter` clause it matches any added enumeration member. For example, if the enumeration members `newday` and `anotherNewDay` have been added to `weekday`, these are equivalent:

```http
$filter=weekday eq unknownFutureValue
$filter=weekday eq newday or weekday eq anotherNewDay
$filter=weekday ge unknownFutureValue
```

If the header is absent, `$filter=weekday` **eq** `unknownFutureValue` matches any new enumeration value. If the header is present, that same filter matches nothing, while `$filter=weekday` **ge** `unknownFutureValue` matches any new enumeration value. (Note: the latter matches new enumeration values whether the header is present or not.)

## SDK code generation

The SDK generates enumeration definitions from the current schema. Requests always include the `include-unknown-enum-members` header.

At runtime, since other members could have been added to the enumeration after code was generated, values unknown to the generated code are translated to `unknownFutureValue`. When the service sees an enumeration property with the value `unknownFutureValue`, it will ignore it and not update the property.

## Resetting an Evolvable Enumeration

Upon a major version change, `unknownFutureValue` can be moved to the end of the enumeration, making known the previously unknown enumeration members.

```csharp
public enum weekday
{
  monday,
  ...
  sunday,
  newday,
  anotherNewDay,
  unknownFutureValue
}
```
