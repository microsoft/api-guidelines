# Property validity using enums

Microsoft Graph API Design Pattern

*There are often cases where a type has a property which denotes a kind of role that the type can fulfill. This property can evolve over time to a point where the way the type that fulfills that role needs to be configured in some way. This pattern allows configuring such properties in a forward compatible way.*

## Problem

When types have these different kinds of roles, they usually end up modeled like this:
```xml
<ComplexType Name="foo">
  <!--ANTI-PATTERN; DO NOT USE-->
  <Property Name="roleKind" Type="graph.roleKind"/>
  <Property Name="roleThirdKindValue" Type="Edm.String"/> <!--only valid when "roleKind" is "third"-->
</ComplexType>

<Enum Name="roleKind">
  <Member Name="first"/>
  <Member Name="second"/>
  <Member Name="third"/>
</Enum>
```

```http
GET https://graph.microsoft.com/v1.0/identity/foos

{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#foo",
  "values": [
    {
      "roleKind": "first",
      "roleThirdKindValue": null
    },
    {
      "roleKind": "third",
      "roleThirdKindValue": "some value"
    }
  ]
}
```

In this scenario, `roleThirdKindValue` being valid depends on what value is provided for `roleKind` (Please note that, although these examples use an enum for `roleKind`, 
other types are still susceptible to this anti-pattern. Please review [this](https://dev.azure.com/msazure/One/_wiki/wikis/Microsoft%20Graph%20Partners/211730/Finite-number-of-choices) 
guidance to determine if an enum better suites your purposes). 

## Solution

When it happens that the validity of one property depends on the value of another property there are two general approaches:
1. Use inheritance
2. Use composition

Please review [these](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/inheritance#inheritance-and-an-is-a-relationship) MSDN guidelines for using inheritance ("is-a" 
relationship) or composition ("has-a" relationship). This guidance will be focused on composition. In that situation, the API should be modeled this way:

```xml
<ComplexType Name="foo">
  <Property Name="roleKind" Type="graph.roleKind"/>
  <Property Name="roleKindSettings" Type="graph.roleKindSettings"/>
</ComplexType>

<Enum Name="roleKind">
  <Member Name="first"/>
  <Member Name="second"/>
  <Member Name="third"/>
</Enum>

<ComplexType Name="roleKindSettings" IsAbstract="true">
</ComplexType>

<ComplexType Name="roleThirdKindSettings" BaseType="graph.roleKindSettings">
  <Property Name="roleThirdKindValue" Type="Edm.String"/>
</ComplexType>
```

```http
GET https://graph.microsoft.com/v1.0/identity/foos

{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#foo",
  "values": [
    {
      "roleKind": "first",
      "roleKindSettings": null
    },
    {
      "roleKind": "third",
      "roleKindSettings": {
        "@odata.type":"microsoft.graph.roleThirdKindSettings",
        "roleThirdKindValue": "some value"
      }
    }
  ]
}
```

The first thing to notice is that the *validity* of a property (`roleThirdKindValue`) is no longer dependent on the value of another property (`roleKind`). This makes the API easier to 
understand without needing to refer to documentation, and also allows the type system to handle error cases syntactically, rather than relying on the workload to implement all of the error
handling explicitly. The *possible* values for `roleThirdKindValue` are still dependent on the value of `roleKind`.

The second thing to notice is that it always gives us a path forward if additional "settings" are needed. For example, to add settings for roleKind.second, this type can be added:

```xml
<ComplexType Name="roleSecondKindSettings" BaseType="graph.roleKindSettings">
  <Property Name="roleSecondKindValue" Type="Edm.String"/>
</ComplexType>
```

```http
GET https://graph.microsoft.com/v1.0/identity/foos

{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#foo",
  "values": [
    {
      "roleKind": "first",
      "roleKindSettings": null
    },
    {
      "roleKind": "third",
      "roleKindSettings": {
        "@odata.type":"microsoft.graph.roleThirdKindSettings",
        "roleThirdKindValue": "some value"
      }
    },
    {
      "roleKind": "second",
      "roleKindSettings": {
        "@odata.type":"microsoft.graph.roleSecondKindSettings",
        "roleSecondKindValue": "some other value"
      }
    }
  ]
}
```

It also doesn't require introducing breaking changes. The original enum property remains, meaning that we only have the additive change of the "settings" property. 

The third thing to notice is that we can support additional "roles" on the same type. Suppose we later add:

```xml
<ComplexType Name="foo">
  <Property Name="roleKind" Type="graph.roleKind"/>
  <Property Name="roleKindSettings" Type="graph.roleKindSettings"/>
  <Property Name="anotherRoleKind" Type="graph.anotherRoleKind"/>
</ComplexType>

<Enum Name="roleKind">
  <Member Name="first"/>
  <Member Name="second"/>
  <Member Name="third"/>
</Enum>

<Enum Name="anotherRoleKind">
  <Member Name="first"/>
  <Member Name="second"/>
  <Member Name="third"/>
</Enum>

<ComplexType Name="roleKindSettings" IsAbstract="true">
</ComplexType>

<ComplexType Name="roleThirdKindSettings" BaseType="graph.roleKindSettings">
  <Property Name="roleThirdKindValue" Type="Edm.String"/>
</ComplexType>
```

```http
GET https://graph.microsoft.com/v1.0/identity/foos

{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#foo",
  "values": [
    {
      "roleKind": "first",
      "roleKindSettings": null,
      "anotherRoleKind": "third"
    },
    {
      "roleKind": "third",
      "roleKindSettings": {
        "@odata.type":"microsoft.graph.roleThirdKindSettings",
        "roleThirdKindValue": "some value"
      },
      "anotherRoleKind": "first"
    },
    {
      "roleKind": "second",
      "roleKindSettings": {
        "@odata.type":"microsoft.graph.roleSecondKindSettings",
        "roleSecondKindValue": "some other value"
      },
      "anotherRoleKind": "second"
    }
  ]
}
```

We have added `anotherRoleKind` without needing to disturb the existing `roleKind` (if we modelled this using inheritance, we would not be able to do this).

## When to use this pattern

This pattern should be used whenever an enum property indicates the validity of other properties within an entity type. It can also be adapted for other use cases where the value of a non-enum property indicates the same.

## Issues and considerations

If possible, the enum property should not be introduced in the first place. However, once it is introduced, it is better to leave it to maintain client backwards compatibility.

## Example

An example of this composition pattern on graph can be found in [this](https://microsoftgraph.visualstudio.com/onboarding/_git/onboarding?path=/reviews/213-Add-data-classification-service-API-to-Graph/api.md&version=GCd23e568a1d3f95edbc563d8e3d36a4ddf2e39f53&line=2169&lineEnd=2169&lineStartColumn=3&lineEndColumn=16&lineStyle=plain&_a=contents) 
API review. In this case, the validity of the `bodyText` property depends on the `scanningState` that is present. Although sometimes this would make sense to model with inheritance 
(again, please refer to the MSDN guidance around "is-a" vs "has-a" relationships), doing so would look something like this:

```xml
<ComplexType Name="attachmentContentProperties" baseType="microsoft.graph.contentProperties" ags:IsHidden="true">  
  <Property Name="sensitiveTypes" Type="Collection(microsoft.graph.discoveredSensitiveType)" />
  <Property Name="currentLabel" Type="microsoft.graph.currentLabel" />
</ComplexType>

<ComplexType Name="unscannableAttachmentContentProperties" baseType="microsoft.graph.attachmentContentProperties" ags:IsHidden="true">  
  <Property Name="bodyText" Type="Edm.String" />
</ComplexType>

<ComplexType Name="scannedAttachmentContentProperties" baseType="microsoft.graph.attachmentContentProperties" ags:IsHidden="true">  
</ComplexType>
```

```http
GET https://graph.microsoft.com/v1.0/identity/foos

{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#foo",
  "values": [
    {
      "roleKind": "first",
      "roleKindSettings": null,
      "anotherRoleKind": "third"
    },
    {
      "roleKind": "third",
      "roleKindSettings": {
        "@odata.type":"microsoft.graph.roleThirdKindSettings",
        "roleThirdKindValue": "some value"
      },
      "anotherRoleKind": "first"
    },
    {
      "roleKind": "second",
      "roleKindSettings": {
        "@odata.type":"microsoft.graph.roleSecondKindSettings",
        "roleSecondKindValue": "some other value"
      },
      "anotherRoleKind": "second"
    }
  ]
}
```

Doing this, however, would prevent the API owner from adding something like an `attachmentType` enum which informs the validity of a property about each attachment type. By using composition,
the API owner is able to model both the `scanningState` *and* a future (currently unknown) `attachmentType` enum, something that could look like:

```xml
<ComplexType Name="attachmentContentProperties" baseType="microsoft.graph.contentProperties" ags:IsHidden="true">  
  <Property Name="sensitiveTypes" Type="Collection(microsoft.graph.discoveredSensitiveType)" />
  <Property Name="currentLabel" Type="microsoft.graph.currentLabel" />
  <Property Name="scanningState" Type="microsoft.graph.attachmentScanningState" />
  <Property Name="scanningStateOutcome" Type="microsoft.graph.attachmentScanningStateOutcome" /> <!--null if scanningState is success-->
  <Property Name="attachmentType" Type="microsoft.graph.attachmentType" />
  <Property Name="attachmentTypeData" Type="microsoft.graph.attachmentTypeData" />
</ComplexType>

<EnumType Name="attachmentScanningState">
  <Member Name="success" />
  <Member Name="unscannable" />
  <Member Name="processingLimitExceeded" />
  <Member Name="unknownFutureValue" />
</EnumType>

<EnumType Name="attachmentType">
  <Member Name="image" />
  ...
  <Member Name="unknownFutureValue" />
</EnumType>

...

<ComplexType Name="attachmentTypeData" ags:IsAbstract="true" ags:IsHidden="true">
</ComplexType>

<ComplexType Name="imageAttachmentData" baseType="microsoft.graph.attachmentTypeData" ags:IsHidden="true">  
  <Property Name="parsedStrings" Type="Collection(Edm.String)" />
</ComplexType>
```
