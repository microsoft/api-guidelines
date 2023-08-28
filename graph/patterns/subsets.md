# Modeling collection subsets

Microsoft Graph API Design Pattern

*The modeling collection subsets pattern is the modeling state associated to a collection that may include all instances, an included subset, an excluded subset, no instances, or any combinations of the preceding items.*

## Problem

A common pattern is to apply a policy or state to a collection of resources. With this, there also comes the question of how to model cases where we want to apply to `all` or `none` without having to special case these values within the collection set or introduce cross-property dependencies. Likewise, we'd like to model it in a way where it is easy to understand and interpret usage from just looking at the schema.

An example is where you have a policy that you need to be able to apply to users in an organization. You might want to support the default **None**, enablement for **All**, or enablement for **Select** users where you only grant it to a few users.

Existing patterns for this either have special-cased strings or have tightly coupled dependencies between two independent properties. Neither is intuitive, both require reading documentation, and neither can be inferred from the schema or within client libraries.

## Solution

Have an abstract base class where all variants of the subset are derived types from the base subset. For more information, see the [general subtyping guidance](./subtypes.md).

The abstract base class may also optionally hold an `enum` for the different variants. If it does, the `enum` must have a member for all possible variants. The purpose of including this is to allow for easier ways to do query and filter operations on variants like `all` and `none` without relying on `isof` functions.

**Base type *without* an enum for the variants**

```xml
    <ComplexType Name="membershipBase" IsAbstract="true" />
```

**Base type *with* an enum for the variants**

```xml
    <ComplexType Name="membershipBase" IsAbstract="true">
      <Property Name="membershipKind" Type="graph.membershipKind"/>
    </ComplexType>

    <EnumType Name="membershipKind">
      <Member Name="all"/>
      <Member Name="enumerated"/>
      <Member Name="none"/>
      <Member Name="unknownFutureValue"/>
    </EnumType>
```

**Derived types**

```xml
    <ComplexType Name="noMembership" BaseType="graph.membershipBase"/>

    <ComplexType Name="allMembership" BaseType="graph.membershipBase"/>

    <ComplexType Name="enumeratedMembership" BaseType = "graph.membershipBase">
      <Property Name="members" Type="Collection(Edm.String)"/>
    </ComplexType>

    <ComplexType Name="excludedMembership" BaseType="graph.membershipBase">
      <Property Name="members" Type="Collection(Edm.String)"/>
    </ComplexType>
```

Be aware that the name values and types in the preceding examples are just examples and can be replaced with your scenario equivalent values. For example, type names don't really need to be `memberships`. The collection doesn't have to be a collection at all; it can be singular and doesn't have to be a string.

These pattern type names should satisfy the following naming conventions:

- The base type name should have the suffix `Base`, and the enumeration type name (if an `enum` is defined) should have the suffix `Kind`.
- Derived child types should have names with enumeration values as the prefixes; for example, if the enumeration member value is `value1`, then the derived type name is `value1<type>`.

```xml
    <ComplexType Name="<type>Base" IsAbstract="true">
      <Property Name="<type>Kind" Type="graph.<type>Kind"/>
    </ComplexType>

    <EnumType Name="<type>Kind">
      <Member Name="<value1>"/>
      <Member Name="<value2>"/>
      <Member Name="unknownFutureValue"/>
    </EnumType>

    <ComplexType Name="value1<type>" BaseType="graph.<type>Base"/>

    <ComplexType Name="value2<type>" BaseType="graph.<type>Base">
      <Property Name="<property-name>" Type="<property-type>"/>
    </ComplexType>
```

## When to use this pattern

Use this pattern when supporting two or more collection states of the following, where at least one of the states is a subset variant:

- All targets
- No targets
- Subset of targets to be included
- Subset of targets to be excluded

If you only ever need to support two states&mdash;All or None&mdash;without using any subsets, it would be better to use a Boolean to toggle on and off.

## Issues and considerations

Given that we are using an overarching subtype model, subtyping model limitations apply here as well; for more details, see the [subtyping documentation](./subtypes.md).

## Example

```http
GET https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies/
```

_Note: Unrelated properties on entities are omitted for easier readability._

```json
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#conditionalAccessPolicy",
    "values": [
        {
            "id": "66d36273-fe4c-d478-dc22-e0179d856ce7",
            "conditions": {
                "users": {
                    "includeGuestsOrExternalUsers": {
                        "externalTenants": {
                            "@odata.type":"microsoft.graph.conditionalAccessAllExternalTenants",
                            "membershipKind": "all"
                        }
                    }
                }
            }
        },
        {
            "id": "99d212f4-d94e-cde1-8e3c-208d78238277",
            "conditions": {
                "users": {
                    "includeGuestsOrExternalUsers": {
                        "externalTenants": {
                            "@odata.type":"microsoft.graph.conditionalAccessEnumeratedExternalTenants",
                            "membershipKind": "enumerated",
                            "members": ["bd005e2a-876d-4bf0-92a1-ae9ff4276d54"]
                        }
                    }
                }
            }
        }
    ]
}
```

```http
POST https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies/
```

_Note: Unrelated properties on entities are omitted for easier readability._

```json
{
    "id": "66d36273-fe4c-d478-dc22-e0179d856ce7",
    "conditions": {
        "users": {
            "includeGuestsOrExternalUsers": {
                "externalTenants": {
                    "@odata.type":"microsoft.graph.conditionalAccessAllExternalTenants"
                }
            }
        }
    }
}
```

or

```http
POST https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies/
```

_Note: Unrelated properties on entities are omitted for easier readability._

```json
{
    "id": "66d36273-fe4c-d478-dc22-e0179d856ce7",
    "conditions": {
        "users": {
            "includeGuestsOrExternalUsers": {
                "externalTenants": {
                    "@odata.type":"microsoft.graph.conditionalAccessEnumeratedExternalTenants",
                    "members": ["bd005e2a-876d-4bf0-92a1-ae9ff4276d54"]
                }
            }
        }
    }
}
```

### Filter when base type has the "kind" enum property

```http
GET https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies?$filter=conditions/users/includeGuestsOrExternalUsers/externalTenants/membershipKind eq 'all'

200 OK
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#conditionalAccessPolicy",
    "values": [
        {
            "id": "66d36273-fe4c-d478-dc22-e0179d856ce7",
            "conditions": {
                "users": {
                    "includeGuestsOrExternalUsers": {
                        "externalTenants": {
                            "@odata.type":"microsoft.graph.conditionalAccessAllExternalTenants",
                            "membershipKind": "all"
                        }
                    }
                }
            }
        }
    ]
}
```

### Filter when base type lacks the "kind" enum property

```HTTP
GET https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies?$filter=isof(conditions/users/includeGuestsOrExternalUsers/externalTenants, microsoft.graph.conditionalAccessAllExternalTenants)

200 OK
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#conditionalAccessPolicy",
    "values": [
        {
            "id": "66d36273-fe4c-d478-dc22-e0179d856ce7",
            "conditions": {
                "users": {
                    "includeGuestsOrExternalUsers": {
                        "externalTenants": {
                            "@odata.type":"microsoft.graph.conditionalAccessAllExternalTenants",
                            "membershipKind": "all"
                        }
                    }
                }
            }
        }
    ]
}
```
