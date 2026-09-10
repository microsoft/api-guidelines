# Revision annotation fidelity

Microsoft Graph API Design Pattern

*Preserve `Org.OData.Core.V1.Revisions` Version and Category metadata exactly when authoring or migrating TypeSpec.*

## Problem

A revision's `Version` is an independent identifier and is not necessarily derived from its `Date`. A Version suffix can also resemble a Category even when the source CSDL omits the `Category` property.

Deriving Version from Date or inferring Category changes the published schema contract.

## Solution

Use the optional `MsGraph.RevisionOptions` argument on `@deprecatedDefinition` and `@privatePreviewDefinition`.

```typespec
@deprecatedDefinition(
  "Removal",
  "Removal",
  "2025-07-01",
  "2026-07-01",
  "Use replacementResource instead.",
  #{ version: "2025-01/Removal", category: null }
)
@privatePreviewDefinition(
  "groundingApi",
  "2025-07-01",
  "2026-07-01",
  "contoso",
  #{ version: "2025-01/PrivatePreview:GroundingAPI" }
)
namespace microsoft.graph {}
```

| Source revision metadata | TypeSpec mapping |
| --- | --- |
| Explicit `Version` | `options.version` with the exact string |
| Category absent on a deprecation | `options.category: null` |
| Category explicitly present | `options.category: "<value>"` |
| No independent Version supplied | Omit `options.version` to retain legacy derivation |

## Issues and considerations

- Do not derive Version from Date when a source Version exists.
- Do not parse or infer Category from the Version suffix.
- Omitting `options.category` on `@deprecatedDefinition` preserves backward-compatible behavior by emitting the positional category.
- `category: null` intentionally suppresses Category emission.
- The options object is optional; existing decorator calls remain valid.
