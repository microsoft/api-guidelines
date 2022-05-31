# Default Properties

Microsoft Graph API Design Pattern

*The default properties pattern allows API producers to omit specific properties from the response unless they are explicitly requested.*

## Problem

--------

API producers may want to control when properties on their entities and complex types are returned, for example if a property is expensive to return, or if a property requires a specific permission.

## Solution

--------

API Producers can use the `ags:Default` metadata annotation to control which properties are default and non-default. When no `ags:Default` annotations are present all properties are default properties.

There are two ways to mark a property as non-default:

1. Apply the `ags:Default="false"` metadata annotation only on the properties that you want to mark as non-default.
2. Apply the `ags:Default="true"` annotation on all properties except the ones that you want to mark as non-default.

### CSDL

```xml
<Schema Namespace="microsoft.graph.example" xmlns="http://docs.oasis-open.org/odata/ns/edm" xmlns:ags="http://aggregator.microsoft.com/internal">
    <EntityType Name="exampleEntity" BaseType="microsoft.graph.entity">
        <Property Name="defaultProperty" Type="Edm.String"/>
        <Property Name="defaultProperty2" Type="Edm.Bool"/>
        <Property Name="nonDefaultProperty" Type="Edm.String" ags:Default="false"/>
    </EntityType>
    <EntityType Name="anotherExampleEntity" BaseType="microsoft.graph.entity">
        <Property Name="defaultProperty" Type="Edm.String" ags:Default="true"/>
        <Property Name="defaultProperty2" Type="Edm.Bool" ags:Default="true"/>
        <Property Name="nonDefaultProperty" Type="Edm.String" />
    </EntityType>
    <ComplexType Name="exampleComplexType">
        <Property Name="defaultProperty" Type="Edm.String"/>
        <Property Name="nonDefaultProperty" Type="Edm.String" ags:Default="false"/>
    </ComplexType>
</Schema>
```

## When to Use this Pattern

--------

This pattern should be used when an API producer has a requirement to omit a property from an API response unless the API consumer explicitly requests the property.

## Issues and Considerations

--------

- API Consumers should clearly document when a property is not returned by default in their API documentation, as at present the public MS Graph metadata does not indicate whether a property is a default property.
- API producers should not mix the use of `ags:Default="false"` and `ags:Default="true"` annotations, For clarity it is recommended to only use the `ags:Default="false"` annotation.
- Unless explicitly marked as not supporting `$filter` and/or `$orderby` a non-default property can be used in a `$filter` or `$orderby` clause even if the property is not included in a `$select` clause
- If there are several related non-default properties the API producer may consider grouping them into a complex type, and then mark the property exposing the complex type as non-default
- Changing a default property to non-default is considered a breaking change.
- Changing a non-default property to default is not a breaking change except in the case where the changed property requires extra permissions.

## Examples

--------

For the following examples we will consider the `managedDevice` entity which contains the `notes` property which is marked as non-default

```xml
<EntityType Name="managedDevice" BaseType="graph.entity">
    <Property Name="displayName" Type="Edm.String" />
    <Property Name="notes" Type="Edm.String" ags:Default="false">
</EntityType>
```

### Default Behavior

```http
GET https://graph.microsoft.com/v1.0/deviceManagement/managedDevices
```

```json
{
    "value": [
        { 
            "id": "0",
            "displayName": "My Laptop"
        },
        { 
            "id": "1",
            "displayName": "Prototype"
        }
    ]
}
```

### Select non-default properties

```http
GET https://graph.microsoft.com/v1.0/deviceManagement/managedDevices?$select=id,displayName,notes
```

```json
{
    "value": [
        { 
            "id": "0",
            "displayName": "My Laptop",
            "notes": "My Surface Laptop"
        },
        { 
            "id": "1",
            "displayName": "Prototype",
            "notes": "Top secret!!!"
        }
    ]
}
```

### Filter on non-default property

```http
GET https://graph.microsoft.com/v1.0/deviceManagement/managedDevices?$filter=notes eq 'Top secret!!!'
```

```json
{
    "value": [
        { 
            "id": "1",
            "displayName": "Prototype"
        }
    ]
}
```

### Sort by non-default property

```http
GET https://graph.microsoft.com/v1.0/deviceManagement/managedDevices?$orderby=notes desc
```

```json
{
    "value": [{ 
            "id": "1",
            "displayName": "Prototype"
        }
        { 
            "id": "0",
            "displayName": "My Laptop"
        }
    ]
}
```
