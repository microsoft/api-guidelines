# Flat bag of properties

Microsoft Graph API Design Pattern

*A known pattern in Microsoft Graph is to model multiple variants of a common concept as a single entity type with all potential properties plus an additional property to distinguish the variants.*

## Problem

API designers need to model a small and limited number of variants of a common concept with a concise list of non-overlapping properties and consistent behavior across variants. The designer also wants to simplify query construction.

## Solution

The API designer creates one entity type with all the potential properties plus an additional property to distinguish the variants, often called `variantType`. For each value of `variantType`, some properties are meaningful and others are ignored.

## When to use this pattern

The flat bag pattern is useful when there is a small number of variants with similar behavior, and variants are queried for mostly read-only operations. The pattern also makes it syntactically easier to query resources by using the OData `$filter` expression because it doesn't require casting.

## Issues and considerations

In general, the flat bag pattern is the least recommended modeling choice because it is weakly typed, and it is difficult to semantically verify targeted resource modifications. However, there are circumstances when query simplicity and a limited number of properties might overweight considerations of a more strongly typed approach.
The pattern is not recommended for a large number of variants and properties because the payload becomes sparsely populated.

You can consider related patterns such as [type hierarchy](./subtypes.md) and [facets](./facets.md).

## Example

A good example for flat bag implementation is the recurrencePattern type on [recurrencePattern](https://docs.microsoft.com/graph/api/resources/recurrencepattern).

The recurrencePattern has six variants expressed as six different values of the `type` property (for example: daily, weekly, ...). The key here is that for each of these values, some properties are meaningful and others are ignored (for example: `daysOfWeek` is relevant when `type` is `weekly` but not when it is `daily`).

```
<EnumType Name="recurrencePatternType">
        <Member Name="daily" Value="0" />
        <Member Name="weekly" Value="1" />
        <Member Name="absoluteMonthly" Value="2" />
        <Member Name="relativeMonthly" Value="3" />
        <Member Name="absoluteYearly" Value="4" />
        <Member Name="relativeYearly" Value="5" />
</EnumType>

<ComplexType Name="recurrencePattern" ags:WorkloadIds="Microsoft.Exchange,Microsoft.IdentityGovernance.AccessReviews,Microsoft.IGAELM,Microsoft.PIM.AzureRBAC,Microsoft.Tasks,Microsoft.Teams.Shifts,Microsoft.Todo" ags:PassthroughWorkloadIds="Microsoft.Exchange">
        <Property Name="dayOfMonth" Type="Edm.Int32" Nullable="false" />
        <Property Name="daysOfWeek" Type="Collection(graph.dayOfWeek)" />
        <Property Name="firstDayOfWeek" Type="graph.dayOfWeek" />
        <Property Name="index" Type="graph.weekIndex" />
        <Property Name="interval" Type="Edm.Int32" Nullable="false" />
        <Property Name="month" Type="Edm.Int32" Nullable="false" />
        <Property Name="type" Type="graph.recurrencePatternType" />
</ComplexType>
```

### TypeSpec representation

The same `recurrencePattern` flat-bag is expressed in TypeSpec using the `@flatBag` decorator on the model plus `@variant` decorators on the variant-conditional sibling properties.
The discriminator property is conventionally named `type`, and the discriminator enum must be closed (no `unknownFutureValue`).

```TypeSpec
enum recurrencePatternType {
  daily: 0,
  weekly: 1,
  absoluteMonthly: 2,
  relativeMonthly: 3,
  absoluteYearly: 4,
  relativeYearly: 5,
}

@flatBag("type")
@complex model recurrencePattern {
  type: recurrencePatternType | null;

  // Shared property — meaningful for every variant
  interval: int32;

  // wire-fidelity: shipped as Nullable="false"; variant-conditional but non-nullable
  @variant("absoluteMonthly", "absoluteYearly") dayOfMonth: int32;

  @variant("weekly", "relativeMonthly", "relativeYearly") daysOfWeek: dayOfWeek[];

  @variant("weekly") firstDayOfWeek: dayOfWeek | null;

  @variant("relativeMonthly", "relativeYearly") index: weekIndex | null;

  // wire-fidelity: shipped as Nullable="false"; variant-conditional but non-nullable
  @variant("absoluteYearly", "relativeYearly") month: int32;
}
```

The `@flatBag` and `@variant` decorators are pure semantic metadata — they do not change the compiled CSDL output, which matches the CSDL shown above.
Their purpose is to enable validation that variant-conditional sibling properties target valid discriminator enum members, and to surface the pattern to downstream tooling.

**Authoring rules:**

- The discriminator property is typed as `<EnumName> | null` to match the shipped Graph CSDL convention.
- Variant-conditional non-collection siblings are nullable (`T | null`).
- Variant-conditional collection siblings use `T[]` rather than `T[] | null` — empty array signals absence per Graph wire convention.
- Properties meaningful for every variant (`interval` above) remain unannotated.
- Non-nullable variant-conditional siblings (like `dayOfMonth`, `month`) preserve the shipped Graph wire shape.
  The inline `// wire-fidelity:` comment documents the exception, and a `#suppress` for `flat-bag-siblings-should-be-nullable` may be needed under stricter rulesets.