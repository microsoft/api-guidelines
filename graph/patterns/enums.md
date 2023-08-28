# Enums

In OData, enums represent a subset of the nominal type they rely on, and are especially useful in cases where certain properties have predefined, limited options.

```xml
<EnumType Name="color">
    <Member Name="Red" Value="0" />
    <Member Name="Green" Value="1" />
    <Member Name="Blue" Value="2" />
</EnumType>
```

## Pros

- Our SDK generators will translate the enum to the best representation of the target programming language, resulting in a better developer experience and free client side validation

## Cons

- Adding a new value requires to go through a (generally fast) API Review
- If the enum is not [evolvable](./patterns/evolvable-enums.md), adding a new value is a breaking change and will generally not be allowed

## Enum or Booleans

Enumerations are a good alternative to Booleans when one of the two values (`true`, `false`) conveys other possible values not yet conceived. Let's assume we have an `publicNotification` type and a property to communicate how to display it:

```xml
<ComplexType Name="publicNotification">
  <Property Name="title" Type="Edm.String" />
  <Property Name="message" Type="Edm.String" />
  <Property Name="displayAsTip" Type="Edm.Boolean" />
</ComplexType>
```

The `false` value here merely communicates that the notification shall not be displayed as a tip. What if, in the future, the notification could be displayed as a `tip` or `alert`, and then in a more distant future, a `dialog` option is viable?

With the current model, the only way is to add more boolean properties to convey the new information:

```diff
<ComplexType Name="publicNotification">
  <Property Name="title" Type="Edm.String" />
  <Property Name="message" Type="Edm.String" />
  <Property Name="displayAsTip" Type="Edm.Boolean" />
+ <Property Name="displayAsAlert" Type="Edm.Boolean" />
+ <Property Name="displayAsDialog" Type="Edm.Boolean" />
</ComplexType>
```

Additionally speaking, the workload will now also have to validate the data structure and make sure that only one of the 3 values is `true`

By using an evolvable enum, instead, all we need to do is to add new members:

```diff
<ComplexType Name="publicNotification">
  <Property Name="title" Type="Edm.String" />
  <Property Name="message" Type="Edm.String" />
+ <Property Name="displayMethod" Type="microsoft.graph.displayMethod" />
-  <Property Name="displayAsTip" Type="Edm.Boolean" />
- <Property Name="displayAsAlert" Type="Edm.Boolean" />
- <Property Name="displayAsDialog" Type="Edm.Boolean" />
</ComplexType>
```

```xml
<EnumType Name="displayMethod">
    <Member Name="tip" Value="0" />
    <Member Name="unknownFutureValue" Value="1" />
    <Member Name="alert" Value="2" />
    <Member Name="dialog" Value="3" />
</EnumType>
```

Similarly speaking, if you find yourself using a `nullable` Enum, that is a indication that maybe what you are trying to model is something that has 3 states and an enum is more appropraite. For instance, let's assume we have a boolean property called `syncEnabled`, where `null` means that the value is undefined and inherited from the general tenant configuration. Instead of modelling like a boolean:

```xml
<Property Name="syncEnabled" Type="Edm.Boolean" Nullable="true"/>
```

An enum not only better conveys the message:

```xml
<EnumType Name="syncState">
    <Member Name="enabled" Value="0" />
    <Member Name="disabled" Value="1" />
    <Member Name="tenantInherit" Value="2" />
    <Member Name="unknownFutureValue" Value="3" />
</EnumType>
```

but it is also open for future scenarios:

```diff
<EnumType Name="syncState">
    <Member Name="enabled" Value="0" />
    <Member Name="disabled" Value="1" />
    <Member Name="tenantInherit" Value="2" />
    <Member Name="unknownFutureValue" Value="3" />
+   <Member Name="groupInherit" Value="4" />
</EnumType>
```

Additionally speaking, depending on the situation, a nullable enum can very likely be avoided by adding a `none` member.

## Flag Enums or Collection of Enums

In case an enum can have multiple values at the same time the tentation is to model the property as a collection of Enums:

```xml
<Property Name="displayMethods" Type="Collection(displayMethod)"/>
```

However, [Flagged Enums](https://docs.oasis-open.org/odata/odata-csdl-xml/v4.01/odata-csdl-xml-v4.01.html#_Toc38530378) can model this use case scenario:

```diff
- <EnumType Name="displayMethod">
+ <EnumType Name="displayMethod" isFlag="true">
-     <Member Name="tip" Value="0" />
+     <Member Name="tip" Value="1" />
-     <Member Name="unknownFutureValue" Value="1" />
+     <Member Name="unknownFutureValue" Value="2" />
-     <Member Name="alert" Value="2" />
+     <Member Name="alert" Value="4" />
-    <Member Name="dialog" Value="3" />
+    <Member Name="dialog" Value="8" />
</EnumType>
```

With such enum, customers can select multiple values in a single field:

`displayMethod = tip | alert`

## Evolvable enums

_The evolvable enums pattern allows API producers to extend enumerated types with new members without breaking API consumers._

### Problem

Frequently API producers want to add new members to an enum type after it is initially published. Some serialization libraries might fail when they encounter members in an enum type that were added after the serialization model was generated. In this documentation, we refer to any added enum members as unknown.

### Solution

The solution is to add a 'sentinel' member named `unknownFutureValue` at the end of the currently known enum members. The API producer then replaces any member that is numerically after `unknownFutureValue` with `unknownFutureValue`.

If an API consumer can handle unknown enum values, the consumer can opt into receiving the unknown enum members by specifying the `Prefer: include-unknown-enum-members` HTTP header in their requests. The API producer then indicates that this preference has been applied by returning the `Preference-Applied: include-unknown-enum-members` HTTP header in the response.

### When to use this pattern

It is a best practice to include an `unknownFutureValue` value when the enum is initially introduced to allow flexibility to extend the enum during the lifetime of the API. Even if the API producer believes that they have included all possible members in an enum, we still strongly recommend that you include an `unknownFutureValue` member to allow for unforeseen future circumstances that may require extending the enum.

This pattern must not be used in scenarios where an API consumer wants to use enum members that are not known to the API producer.

### Issues and considerations

Consider the following:

- An enum member with the name of `unknownFutureValue` MUST only be used as a sentinel value. An API producer MUST not include a member named `unknownFutureValue` in an enum for any other purpose.

- Changing the value (that is, position) of the `unknownFutureValue` sentinel member is considered a breaking change and must follow the [deprecation](../deprecation.md) process.

- Enum types can have multiple members with the same numeric value to allow for aliasing enum members. `unknownFutureValue` MUST not be aliased to any other enum member.

- There is no ability for a client to indicate that it can handle a subset of unknown enum members. Instead, they can only specify that either they cannot handle any unknown enum members or they can handle any unknown enum members.

- The `Prefer: include-unknown-enum-members` header applies to all included enums in the request/response. There is no way for an API consumer to apply the behavior to only a subset of enum types.

- New values MUST not be inserted into the enum before `unknownFutureValue`. Implementers are recommended to make the numeric value of `unknownFutureValue` one greater than the last known enum member to ensure that there are no gaps into which a new member could be inadvertently added. The exception to this is the case of flagged enums, in which case the value of `unknownFutureValue` should be the next power of 2 value.

- For flagged enums, care should be exercised to ensure that `unknownFutureValue` is not included in any enum members that represent a combination of other enum members.

- If the value of a property containing a flag enum contains multiple unknown values, they should all be replaced with a single `unknownFutureValue` value (that is, there should not be multiple `unknownFutureValue` values returned).

- If an API consumer specifies `unknownFutureValue` for the value of a property in a `POST`/`PUT` request or as a parameter of an action or function, the API producer must reject the request with a `400 Bad Request` HTTP status.

- If an API consumer specifies `unknownFutureValue` for the value of a property in a `PATCH` request, the API producer must treat the property as if it were absent (that is, the existing value should not be changed). In the case where the API producer treats `PATCH` as an upsert, the call MUST be rejected with a `400 Bad Request` HTTP status.

- If an API consumer specifies an enum member greater than `unknownFutureValue` in any request without specifying the `Prefer: include-unknown-enum-members` header, the API producer must reject the request with a `400 Bad Request` HTTP status.

- For details about how the `unknownFutureValue` value is handled as part of a `$filter` clause, consult the following examples:

  - **CSDL**

    ```xml
        <EntityType Name="example">
            <Property Name="enumProperty" Type="exampleEnum"/>
        </EntityType>

        <EnumType Name="exampleEnum">
            <Member Name="default" Value="0"/>
            <Member Name="one" Value="1"/>
            <Member Name="unknownFutureValue" Value="2"/>
            <Member Name="newValue" Value="3"/>
        </EnumType>
    ```

  - **Filter behavior**

    | `$filter` clause                     | `Prefer: include-unknown-enum-members` Absent                                                                                       | `Prefer: include-unknown-enum-members` Present                                     |
    | ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
    | `enumProperty eq unknownFutureValue` | Return entities where enumProperty has any value greater than `unknownFutureValue` replacing actual value with `unknownFutureValue` | Return nothing                                                                     |
    | `enumProperty gt unknownFutureValue` | Return entities where enumProperty has any value greater than `unknownFutureValue` replacing actual value with `unknownFutureValue` | Return entities where enumProperty has any value greater than `unknownFutureValue` |
    | `enumProperty lt unknownFutureValue` | Return entities where enumProperty has any known value (i.e. less than `unknownFutureValue`)                                        | Return entities where enumProperty has any value less than `unknownFutureValue`    |
    | `enumProperty eq newValue`           | `400 Bad Request`                                                                                                                   | Return entities where enumProperty has the value `newValue`                        |
    | `enumProperty gt newValue`           | `400 Bad Request`                                                                                                                   | Return entities where enumProperty has a value greater than `newValue`             |
    | `enumProperty lt newValue`           | `400 Bad Request`                                                                                                                   | Return entities where enumProperty has a value less than `newValue`                |

- If an evolvable enum is included in an `$orderby` clause, the actual numeric value of the member should be used to order the collection. After sorting, the member should then be replaced with `unknownFutureValue` when the `Prefer: include-unknown-enum-members` header is absent.

### Examples

For the following examples, we consider the `managedDevice` entity, which refers to the `managedDeviceArchitecture` enum type.

```xml
<!-- Simplified entity for example purposes -->
<EntityType Name="managedDevice" BaseType="graph.entity">
    <Property Name="displayName" Type="Edm.String" />
    <Property Name="processorArchitecture" Type="graph.managedDeviceArchitecture"/>
</EntityType>
```

When the `managedDeviceArchitecture` enum was initially published to Microsoft Graph, it was defined as follows:

```xml
<!-- Slightly modified enum for example purposes -->
<EnumType Name="managedDeviceArchitecture">
    <Member Name="unknown" Value="0"/>
    <Member Name="x86" Value="1"/>
    <Member Name="x64" Value="2"/>
    <Member Name="arm" Value="3"/>
    <Member Name="arm64" Value="4"/>
    <Member Name="unknownFutureValue" Value="5"/>
</EnumType>
```

The enum was later extended to add a new value of `quantum`, leading to the following CSDL:

```xml
<EnumType Name="managedDeviceArchitecture">
    <Member Name="unknown" Value="0"/>
    <Member Name="x86" Value="1"/>
    <Member Name="x64" Value="2"/>
    <Member Name="arm" Value="3"/>
    <Member Name="arm64" Value="4"/>
    <Member Name="unknownFutureValue" Value="5"/>
    <Member Name="quantum" Value="6"/>
</EnumType>
```

#### Default behavior

```http
GET https://graph.microsoft.com/v1.0/deviceManagement/managedDevices?$select=displayName,processorArchitecture
```

```json
{
    "value": [
        {
            "id": "0",
            "displayName": "Surface Pro X",
            "processorArchitecture" : "arm64"
        },
        {
            "id": "1",
            "displayName": "Prototype",
            "processorArchitecture": "unknownFutureValue"
        }
        {
            "id": "2",
            "displayName": "My Laptop",
            "processorArchitecture": "x64"
        }
    ]
}
```

In this case, the value of the `processorArchitecture` property is `quantum`. However, because the client did not request the `include-unknown-enum-members` header, the value was replaced with `unknownFutureValue`.

#### Include opt-in header

```http
GET https://graph.microsoft.com/v1.0/deviceManagement/managedDevices?$select=displayName,processorArchitecture

Prefer: include-unknown-enum-members
```

```json
Preference-Applied: include-unknown-enum-members

{
    "value": [
        {
            "displayName": "Surface Pro X",
            "processorArchitecture" : "arm64"
        },
        {
            "displayName": "Prototype",
            "processorArchitecture": "quantum"
        },
        {
            "displayName": "My Laptop",
            "processorArchitecture": "x64"
        }
    ]
}
```

#### Default sort behavior

```http
GET https://graph.microsoft.com/v1.0/deviceManagement/managedDevices?$select=displayName,processorArchitecture&$orderBy=processorArchitecture
```

```json
{
  "value": [
    {
      "displayName": "Surface Pro X",
      "processorArchitecture": "arm64"
    },
    {
      "displayName": "My Laptop",
      "processorArchitecture": "x64"
    },
    {
      "displayName": "Prototype",
      "processorArchitecture": "unknownFutureValue"
    }
  ]
}
```

#### Sort behavior with opt-in header

```http
GET https://graph.microsoft.com/v1.0/deviceManagement/managedDevices?$select=displayName,processorArchitecture

Prefer: include-unknown-enum-members
```

```json
Preference-Applied: include-unknown-enum-members

{
    "value": [
        {
            "displayName": "Surface Pro X",
            "processorArchitecture" : "arm64"
        },
        {
            "displayName": "My Laptop",
            "processorArchitecture": "x64"
        },
        {
            "displayName": "Prototype",
            "processorArchitecture": "quantum"
        }
    ]
}
```

#### Default filter behavior

```http
GET https://graph.microsoft.com/v1.0/deviceManagement/managedDevices?$select=displayName,processorArchitecture&$filter=processorArchitecture gt x64
```

```json
{
  "value": [
    {
      "displayName": "My Laptop",
      "processorArchitecture": "x64"
    },
    {
      "displayName": "Prototype",
      "processorArchitecture": "unknownFutureValue"
    }
  ]
}
```

#### Filter behavior with opt-in header

```http
GET https://graph.microsoft.com/v1.0/deviceManagement/managedDevices?$select=displayName,processorArchitecture&$filter=processorArchitecture gt x64

Prefer: include-unknown-enum-members
```

```json
Preference-Applied: include-unknown-enum-members

{
    "value": [
        {
            "displayName": "My Laptop",
            "processorArchitecture": "x64"
        },
        {
            "displayName": "Prototype",
            "processorArchitecture": "quantum"
        }
    ]
}
```

#### Patch example

```http
PATCH https://graph.microsoft.com/v1.0/deviceManagement/managedDevices/1

{
    "displayName": "Secret Prototype",
    "processorArchitecture": "unknownFutureValue"
}
```

```json
{
  "id": "1",
  "displayName": "Secret Prototype",
  "processorArchitecture": "unknownFutureValue"
}
```

```http
GET https://graph.microsoft.com/v1.0/deviceManagement/managedDevices/1
Prefer: include-unknown-enum-members
```

```json
Preference-Applied: include-unknown-enum-members

{
    "id": "1",
    "displayName": "Secret Prototype",
    "processorArchitecture": "quantum"
}
```

### Flag enum examples

For the following examples, we consider the `windowsUniversalAppX` entity, which refers to the `windowsArchitecture` flag enum type.

```xml
<!-- Simplified entity for example purposes -->
<EntityType Name="windowsUniversalAppX" BaseType="graph.entity">
    <Property Name="displayName" Type="Edm.String" />
    <Property Name="applicableArchitectures" Type="graph.windowsArchitecture"/>
</EntityType>
```

When the `windowsArchitecture` enum was initially published to Microsoft Graph, it was defined as follows:

```xml
<!-- Slightly modified enum for example purposes -->
<EnumType Name="windowsArchitecture" IsFlags="true">
    <Member Name="none" Value="0"/>
    <Member Name="x86" Value="1"/>
    <Member Name="x64" Value="2"/>
    <Member Name="arm" Value="4"/>
    <Member Name="neutral" Value="8"/>
    <Member Name="unknownFutureValue" Value="16" />
</EnumType>
```

The enum was later extended to add a new value of `quantum`, leading to the following CSDL:

```xml
<EnumType Name="windowsArchitecture" IsFlags="true">
    <Member Name="none" Value="0"/>
    <Member Name="x86" Value="1"/>
    <Member Name="x64" Value="2"/>
    <Member Name="arm" Value="4"/>
    <Member Name="neutral" Value="8"/>
    <Member Name="unknownFutureValue" Value="16" />
    <Member Name="quantum" Value="32" />
</EnumType>
```

### Flag enum default behavior

```http
GET https://graph.microsoft.com/v1.0/deviceAppManagement/mobileApps?$select=displayName,applicableArchitectures
```

```json
{
    "value": [
        {
            "id": "0",
            "displayName": "OneNote",
            "applicableArchitectures" : "neutral"
        },
        {
            "id": "1",
            "displayName": "Minecraft",
            "applicableArchitectures": "x86,x64,arm,unknownFutureValue"
        }
        {
            "id": "2",
            "displayName": "Edge",
            "applicableArchitectures": "x64,arm,unknownFutureValue"
        }
    ]
}
```

In this case, the value of the `applicableArchitectures` property includes `quantum`. However, because the client did not request the `include-unknown-enum-members` header, the value was replaced with `unknownFutureValue`.

### Flag enum include opt-in header

```http
GET https://graph.microsoft.com/v1.0/deviceAppManagement/mobileApps?$select=displayName,applicableArchitectures

Prefer: include-unknown-enum-members
```

```json
Preference-Applied: include-unknown-enum-members

{
    "value": [
        {
            "id": "0",
            "displayName": "OneNote",
            "applicableArchitectures" : "neutral"
        },
        {
            "id": "1",
            "displayName": "Minecraft",
            "applicableArchitectures": "x86,x64,arm,quantum"
        }
        {
            "id": "2",
            "displayName": "Edge",
            "applicableArchitectures": "x64,arm,quantum"
        }
    ]
}
```

#### Flag enum default filter behavior

```http
GET https://graph.microsoft.com/v1.0/deviceAppManagement/mobileApps?$select=displayName,applicableArchitectures&$filter=applicableArchitectures has unknownFutureValue
```

```json
{
    "value": [
        {
            "id": "1",
            "displayName": "Minecraft",
            "applicableArchitectures": "x86,x64,arm,unknownFutureValue"
        }
        {
            "id": "2",
            "displayName": "Edge",
            "applicableArchitectures": "x64,arm,unknownFutureValue"
        }
    ]
}
```

#### Flag enum include opt-in header filter behavior

```http
GET https://graph.microsoft.com/v1.0/deviceAppManagement/mobileApps?$select=displayName,applicableArchitectures&$filter=applicableArchitectures has unknownFutureValue

Prefer: include-unknown-enum-members
```

```json
Preference-Applied: include-unknown-enum-members

{
    "value": []
}
```

#### Flag enum patch example

```http
PATCH https://graph.microsoft.com/v1.0/deviceAppManagement/mobileApps/1

{
    "displayName": "Minecraft 2",
    "processorArchitecture": "unknownFutureValue"
}
```

```json
{
  "id": "1",
  "displayName": "Minecraft 2",
  "applicableArchitectures": "unknownFutureValue"
}
```

```http
GET https://graph.microsoft.com/v1.0/deviceAppManagement/mobileApps/1

Prefer: include-unknown-enum-members
```

```json
Preference-Applied: include-unknown-enum-members

{
    "id": "1",
    "displayName": "Minecraft 2",
    "applicableArchitectures": "x86,x64,arm,quantum"
}
```
