# Capability restrictions

Microsoft Graph API Design Pattern

*Declare the HTTP methods supported by a structured type in the published metadata.*

## Problem

Some entity and complex types support only a specific set of HTTP methods. Without
machine-readable metadata, clients and tooling cannot discover those restrictions
from the API description.

The `Org.OData.Capabilities.V1` vocabulary does not define a
`SupportedHttpMethods` term, so using that namespace would produce an unresolved
CSDL term.

## Solution

Apply `@supportedHttpMethods` to an `@entity` or `@complex` model. List the
supported HTTP method tokens in the order they should be emitted.

```typespec
@open
@supportedHttpMethods("GET", "PATCH")
@complex
model mutablePayload {
  value?: string;
}
```

The decorator has the following contract:

```typespec
extern dec supportedHttpMethods(
  target: Model,
  method: valueof string,
  ...otherMethods: valueof string[]
);
```

The first method is required. Additional methods are optional and preserve their
declared order. Standard and extension methods are supported when they use valid
HTTP token characters.

| Decorator | Pattern | Target | Purpose |
| --- | --- | --- | --- |
| `@supportedHttpMethods` | capability-restrictions | Model | Declares the ordered HTTP methods supported by an `@entity` or `@complex` model. |

## Compiled CSDL

The TypeSpec example emits a workload-owned term in the public namespace and
references that term from the complex type:

```xml
<ComplexType Name="mutablePayload" OpenType="true">
  <Property Name="value" Type="Edm.String" Nullable="false"/>
  <Annotation Term="microsoft.graph.test.SupportedHttpMethods" ags:IsRemovable="true">
    <Collection>
      <String>GET</String>
      <String>PATCH</String>
    </Collection>
  </Annotation>
</ComplexType>
<Term Name="SupportedHttpMethods" Type="Collection(Edm.String)">
  <Annotation
    Term="Org.OData.Core.V1.Description"
    String="HTTP methods supported by an entity or complex type."
    ags:IsRemovable="true"/>
</Term>
```

The term QName uses the workload's public namespace. The annotation value is a
`Collection` of `String` values in the same order as the decorator arguments,
and the matching term type is `Collection(Edm.String)`.

## When to use this pattern

Use this pattern when an entity or complex type supports a defined set of HTTP
methods that clients and tooling need to discover from metadata.

## Issues and considerations

- Apply the decorator only to an `@entity` or `@complex` model.
- Specify at least one HTTP method.
- Use valid HTTP token strings. Do not include spaces or separators that are not
  allowed in an HTTP method token.
- Do not repeat a method in the same decorator or across repeated applications
  to the same model.
- Preserve the intended method order because the emitter preserves the declared
  order in CSDL.
- Do not define the term in `Org.OData.Capabilities.V1`; that vocabulary does not
  contain `SupportedHttpMethods`.
