### Enums

In OData, enums represent a subset of the nominal type they rely on, and are especially useful in cases where certain properties have predefined, limited options.

```xml
<EnumType Name="color">
    <Member Name="Red" Value="0" />
    <Member Name="Green" Value="1" />
    <Member Name="Blue" Value="2" />
</EnumType>
```

#### Pros

- Our SDK generators will translate the enum to the best representation of the target programming language, resulting in a better developer experience and free client side validation

#### Cons

- Adding a new value requires to go through a (generally fast) API Review
- If the enum is not [evolvable](./patterns/evolvable-enums.md), adding a new value is a breaking change and will generally not be allowed

#### Enum or Booleans

Enumerations are a good alternative to Booleans when one of the two values (`true`, `false`) conveys other possible values not yet conceived. Let's assume we have an `Error` type and a property to communicate how to display it:

```xml
<ComplexType Name="error">
  <Property Name="title" Type="Edm.String" />
  <Property Name="message" Type="Edm.String" />
  <Property Name="displayAsTip" Type="Edm.Boolean" />
</ComplexType>
```

The `false` value here merely communicates that the error shall not be displayed as a tip. What if, in the future, the error could be displayed as a `tip` or `alert`, and then in a more distant future, a `dialog` option is viable?

With the current model, the only way is to add more boolean properties to convey the new information:

```diff
<ComplexType Name="error">
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
<ComplexType Name="error">
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

#### Flag Enums or Collection of Enums

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
