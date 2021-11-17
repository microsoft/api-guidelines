# Pattern Name

Microsoft Graph API Design Pattern



Modeling state associated to a collection that may include all instances, a included subset, an excluded subset, no instances, or any combinations of the above.

## Context

It is a common pattern to have to have a need to apply a policy or state to a collection of objects. With this, there also comes the question of how to model cases where we want to apply to 'all' or 'none' as well without having to special case these values within the collection set or introducing cross property dependencies. Likewise we'd like to model it in a way where it is easy to understand and interpret usage from just looking at the schema.

A simple example of something like this would be say you have some policy you need to be able to apply to users in an organization. You may want to support default **None**, enablement for **All** or enablement for **Select** users where you only grant it to a handful of users manually.

## Problem
--------

Existing patterns for this require us to either have special cased 'strings' or have tightly coupled depedencies between two independent properties. Neither are intuitive, require reading documentation and cannot be inferred from the schema.

## Solution
--------

Have an abstract base class the upper level container will point to and all 'flavors' of the subset are then derived types from the base subset. You can find general subtyping guidance [here](https://github.com/microsoft/api-guidelines/blob/op-graphPatterns/graph/Modelling%20with%20Subtypes%20Pattern.md).

Base Type
```xml
    <ComplexType Name="membership" IsAbstract="true"/>
```

Derived Types
```xml
    <ComplexType Name="noMembership" BaseType="graph.membership"/>

    <ComplexType Name="allMembership" BaseType="graph.membership"/>

    <ComplexType Name="enumeratedMembership" BaseType = "graph.membership">
      <Property Name="members" Type="Collection(Edm.String)"/>
    </ComplexType>

    <ComplexType Name="excludedMembership" BaseType = "graph.membership">
      <Property Name="members" Type="Collection(Edm.String)"/>
    </ComplexType>
```

## Issues and Considerations
-------------------------

Given we are using overarching subtype model. The subtyping model limitations apply here as well, see [subtyping documentation](https://github.com/microsoft/api-guidelines/blob/op-graphPatterns/graph/Modelling%20with%20Subtypes%20Pattern.md) for more details.
 

## When to Use this Pattern
------------------------

Pattern is to be used when desire is to support two or more collection states of the following where at least one of the states is a subset flavor.
- All targets
- No targets
- Subset of targets to be included
- Subset of targets to be excluded.

If you only need to support two states - All or None - without usage of any subsets, it would be better to use the boolean model to toggle on an off.

## Example
-------

GET https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies/

_Note: unrelated properties on entities omitted for easier readiability_

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
```

 

 
