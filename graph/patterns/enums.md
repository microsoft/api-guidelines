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

If used, `EnumType` names should be singular if the are non-flags enums, and the names should be plural if they are flags enums.


#### Nullable enums

Enum properties can be marked `Nullable="true"`, which means the property can hold `null` in addition to any defined member.
Before making an enum property nullable, consider whether a sentinel member like `none` better communicates the intent.

##### `null` vs `none`

| Value | Meaning | Use when |
|---|---|---|
| `null` | The property has no value — it was never set or is not applicable | The absence of a value is semantically different from every defined member |
| `none` | An explicit "nothing selected" choice within the enum's domain | "No selection" is a valid, intentional state the caller can set |

> **Note:** The `unknownFutureValue` sentinel is always required as the last known member of every enum (see [evolvable enums](./evolvable-enums.md)).
> It is unrelated to nullability and must be present regardless of whether the property is nullable or uses a `none` member.

##### Prefer a `none` member over nullable

In most cases, add a `none` member (value `0`) instead of making the property nullable.
This keeps the property non-nullable, which is simpler for SDK consumers and avoids the three-way ambiguity of "is it null, is it none, or is it a real value?"

```xml
<!-- ✅ RECOMMENDED — explicit 'none' member -->
<EnumType Name="priority">
    <Member Name="none" Value="0"/>
    <Member Name="low" Value="1"/>
    <Member Name="normal" Value="2"/>
    <Member Name="high" Value="3"/>
    <Member Name="unknownFutureValue" Value="4"/>
</EnumType>

<Property Name="priority" Type="microsoft.graph.priority" Nullable="false"/>
```

##### When nullable is appropriate

Use `Nullable="true"` on an enum property only when **all** of the following apply:

1. **Absence is meaningful** — `null` represents "not set" or "not applicable," which is semantically distinct from every enum member including a hypothetical `none`.
2. **A sentinel member would be misleading** — adding `none` would imply the caller actively chose "nothing," but the actual semantics are that the property doesn't apply to this instance.
3. **The property is optional on creation** — the service does not assign a default value; `null` is the expected state until the caller explicitly sets one.

```xml
<!-- Acceptable — null means "not yet evaluated" which is distinct from any severity level -->
<Property Name="severity" Type="microsoft.graph.severity" Nullable="true"/>
```

##### Anti-pattern: nullable + `none`

Do **not** combine a nullable enum with a `none` member.
This creates two ways to express "no value" and forces callers to handle both `null` and `none`, leading to inconsistency and bugs.

```xml
<!-- ❌ WRONG — ambiguous: is "no value" null or none? -->
<EnumType Name="priority">
    <Member Name="none" Value="0"/>
    <Member Name="low" Value="1"/>
    <Member Name="high" Value="2"/>
    <Member Name="unknownFutureValue" Value="3"/>
</EnumType>

<Property Name="priority" Type="microsoft.graph.priority" Nullable="true"/>
```

Pick one: either `none` with `Nullable="false"`, or no `none` member with `Nullable="true"`.

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

In cases where two properties want to use the same *conceptual* `EnumType`, but one property is a collection while the other is single-values, the model should define *two* separate `EnumType`s, one being a non-flags enum with a singular name and the other marked as a flags enum with its name being the plural form of the non-flags enum.

#### Flag enum + non-flag enum

There are occasions where one API will want to use a non-flag enum, but another API will want a flags enum. 
For example, the `displayMethod` example above may have one API that is configuring which display methods to use, and another API which is configuring that particular display method. 
In this case, the first API will want a flags enum, but the second API will want to only allow configuring one display method at a time, and will therefore prefer a non-flags enum.

Two enum types should be defined, one as a flags enum and the other as a non-flags enum.
The flags enum should be named such that it is plural, and the non-flags enum should be named such that it is singular.
The two types should be kept in sync with each other.
