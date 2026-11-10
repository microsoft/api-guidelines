# Custom terms and instance annotations

Microsoft Graph API Design Pattern

*Define custom instance-annotation terms without losing their CSDL applicability, documentation semantics, or AGS metadata.*

## Problem

An API needs a custom instance annotation. The term contract can include more than its name and type: it can restrict where the term applies, use long-form documentation, and carry AGS attributes such as `ags:IsHidden`.

Dropping this metadata during TypeSpec authoring or CSDL migration changes the published schema contract.

## Solution

Declare the term with the namespace-level `@term` decorator. Use its options object to preserve optional CSDL metadata.

```typespec
@publicNamespace("microsoft.graph")
@workloadNamespace("contoso.example.terms")
@term(
  "SyntheticTerm",
  string,
  "Synthetic long-form documentation.",
  #{
    appliesTo: "Property Term",
    descriptionKind: "LongDescription",
    agsAttributes: #{
      IsHidden: "true"
    }
  }
)
namespace termExample {}
```

The TypeSpec emits:

```xml
<Term Name="SyntheticTerm" Type="Edm.String" AppliesTo="Property Term" ags:IsHidden="true">
  <Annotation Term="Org.OData.Core.V1.LongDescription"
      String="Synthetic long-form documentation." ags:IsRemovable="true"/>
</Term>
```

## When to use this pattern

Use this pattern when a workload defines a custom instance annotation or migrates a `<Term>` declaration from existing CSDL.

The four-argument `@term(name, type, description)` form is sufficient when the term has no `AppliesTo` or AGS attributes and uses `Org.OData.Core.V1.Description`.

## Issues and considerations

- Preserve the exact space-separated `AppliesTo` value from the source CSDL.
- Use `descriptionKind: "LongDescription"` only for `Org.OData.Core.V1.LongDescription`.
- Specify AGS attribute names without the `ags:` prefix; the emitter adds the namespace prefix.
- AGS attribute values are strings because they are emitted as XML attributes.
- Do not substitute prose documentation for metadata required by the schema contract.
