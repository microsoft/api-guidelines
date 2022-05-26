# Namespace

Microsoft Graph API Design Pattern

*The namespace pattern provides the ability to organize resource definitions together into a logical set.*

## Problem

When building a complex offering, API designers might need to model many different
resources and their relationships. For a better user experience and
discoverability, related API elements need to be grouped together.  

## Solution

API designers can use the namespace attribute of the CSDL schema to declare a
namespace and logically organize related API entities in the Microsoft Graph metadata.

```XML
<Schema Namespace="microsoft.graph.{namespace}">
...
</Schema>
```

A public namespace must contain the `microsoft.graph.` prefix and be presented in camel
case; that is, `microsoft.graph.myNamespace`.

When type casting is required in the API query, request, or response, a fully
qualified type name is represented as concatenation of a namespace and a type
name. For a consistent user experience, namespaces MUST be aligned with the corresponding API category path segment.

## When to use this pattern

API resource grouping creates a user-friendly experience, keeping all resources for a specific feature close together and limiting the length of IDE prompts such as auto-complete in some programming languages.

We recommend that a new namespace be aligned with a top-level API category.

## Issues and considerations

- Microsoft Graph consistency requirements discourage using the same type names for different concepts even within different namespaces. Microsoft Graph type names must be descriptive and unique within the API surface without requiring  full qualification.

- A namespace must be consistent with an API category in the navigation path according to [Microsoft Graph REST API Guidelines](GuidelinesGraph.md#uniform-resource-locators-urls).

- When type name is ambiguous and requires a namespace qualifier, changing a namespace is a breaking change.

- To extend a type in a different schema, a service must declare that schema and the type in it. This is conceptually similar to .NET partial types.

- To reference a type in a different schema, simply refer to that type by its fully qualified name (namespace + type name).

- Cyclical references between namespaces are not allowed because many object-oriented languages don’t support cycles between namespaces.

- Microsoft Graph has some predefined constraints for declared namespaces:

  - All public namespaces must have the prefix `microsoft.graph`.
    
  - Only one level of nesting deeper than `microsoft.graph` is supported.
    
  - If a namespace does not begin with the `microsoft.graph` prefix, all types in the schema are coerced into the main `microsoft.graph` namespace.

## Examples

### Namespace and type declarations

```XML
<Schema Namespace="microsoft.graph.search" xmlns=”<http://docs.oasis-open.org/odata/ns/edm>”\>
…
    <EntityType Name="bookmark" …
    </EntityType>
…
</Schema>
```

Fully qualified type name: `microsoft.graph.search.bookmark`

### Managing multiple schemas

Workloads must define schemas in their CSDL by using the Edmx format.
Following is an example of a workload that exposes multiple namespaces.

> **Tip:** As with schemas that exist in the `microsoft.graph` namespace, defining an
entity type is optional; by default your schema derives all entity types
from `microsoft.graph.entity`.

> **Warning:** Do not deviate from the general structure in the following example.
The schema validation tool expects the XML structure (including XML namespace
declarations) to match this example.

```XML
<?xml version="1.0" encoding="utf-8"?>
<edmx:Edmx Version="4.0" xmlns:edmx="http://docs.oasis-open.org/odata/ns/edmx" xmlns:odata="http://schemas.microsoft.com/oDataCapabilities">
  <edmx:DataServices>
    <Schema Namespace="microsoft.graph.callRecords" xmlns="http://docs.oasis-open.org/odata/ns/edm" xmlns:odata="http://schemas.microsoft.com/oDataCapabilities">
      <EntityType Name="callRecord">
        <Key>
          <PropertyRef Name="id" />
        </Key>
        <Property Name="version" Type="Edm.Int64" Nullable="false" />
        <Property Name="id" Type="Edm.String" Nullable="false" />
      </EntityType>
    </Schema>
    <Schema Namespace="microsoft.graph" xmlns="http://docs.oasis-open.org/odata/ns/edm">
      <EntityContainer Name="defaultContainer">
        <Singleton Name="communications" Type="microsoft.graph.cloudCommunications" />
      </EntityContainer>
      <EntityType Name="cloudCommunications">
        <Key>
          <PropertyRef Name="id" />
        </Key>
        <Property Name="id" Type="Edm.String" Nullable="false" />
        <NavigationProperty Name="callRecords" Type="Collection(microsoft.graph.callRecords.callRecord)" ContainsTarget="true" />
      </EntityType>
    </Schema>
  </edmx:DataServices>
</edmx:Edmx>
```
