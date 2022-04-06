# Namespace

Microsoft Graph API Design Pattern

### *The Namespace provides the ability to organize resource definitions together into a logical set.*

## Problem

When building a complex offering API designers may need to model many different
resources and their relationships. For better user experience and
discoverability related API elements need to be grouped together.  


## Solution

API designers can use the Namespace attribute of the CSDL schema to declare a
namespace and logically organize related API entities in the Graph metadata.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ XML
<Schema Namespace="microsoft.graph.{namespace}">
...
</Schema>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A public namespace must have "microsoft.graph.” prefix and be presented in camel
case, i.e microsoft.graph.myNamespace.

When type casting is required in the API query, request or response, a fully
qualified type name is represented as concatenation of a namespace and a type
name. For consistent user experience namespaces MUST be aligned with the corresponding API category path segment.


## When to Use this Pattern

API resource grouping creates a user-friendly experience keeping all resources
for a specific feature close together. It also allows generating smallersmaller downloadable client components 
and limits the length of IDE prompts such as Intellisense.

We recommend creating a new namespace when a new top-level API category is
introduced.

## Issues and Considerations

1.  Microsoft Graph consistency requirements discourage using the same type
    names for different concepts even within different namespaces. Microsoft
    Graph type names must be descriptive and unique within the API
    surface without requiring  full qualification.

2.  Namespace must be consistent with API category in the navigation path according to [Microsoft Graph REST API Guidelines](https://github.com/microsoft/api-guidelines/blob/vNext/graph/GuidelinesGraph.md#uniform-resource-locators-urls)

3.  When type name is ambiguous and requires a namespace qualifier, changing
    namespace is a breaking change.

4.  To extend a type in a different schema, a service must declare that schema
    and the type in it. This is conceptually similar to .NET partial types.

5.  To reference a type in a different schema, simply refer to that type by
    fully qualified name (namespace + type name).

6.  Cyclical references between namespaces are not allowed as many
    object-oriented languages don’t support a cycle in inheritance.

7.  Microsoft Graph has heuristic rules for declared namespaces:

    1.  All public namespaces must have a prefix ‘microsoft.graph’

    2.  If a namespace does not begin with ‘microsoft.graph’ prefix, all types
        in the schema will be coerced into the main ‘microsoft.graph’ namespace.

## Examples

### Namespace and type declarations:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ XML
<Schema Namespace="microsoft.graph.search" xmlns=”<http://docs.oasis-open.org/odata/ns/edm>”\>
…
    <EntityType Name="bookmark" …
    </EntityType>
…
</Schema>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Fully qualified type name: microsoft.graph.search.bookmark

### Managing multiple schemas:

Workloads must define schemas in their csdl using the Edmx format.
[Microsoft.IC3.DataPlatform](https://dev.azure.com/msazure/One/_git/AD-AggregatorService-Workloads?path=%2FWorkloads%2FMicrosoft.IC3.DataPlatform%2Foverride%2Fschema-Prod-beta.csdl)
is an example of a workload that exposes multiple namespaces.

**Tip:** As with schemas that exist in the microsoft.graph namespace, defining an
entity type is optional, AGS will transform your schema to make all entity types
derive from microsoft.graph.entity.

**Warning:** Do not deviate from the general structure in the example below.
Schema validation tool expects the XML structure (including xml namespace
declarations) to match the example below. 
```XML
<?xml version="1.0" encoding="utf-8"?>
<edmx:Edmx Version="4.0" xmlns:edmx="http://docs.oasis-open.org/odata/ns/edmx" xmlns:ags="http://aggregator.microsoft.com/internal" xmlns:odata="http://schemas.microsoft.com/oDataCapabilities">
  <edmx:DataServices>
    <Schema Namespace="microsoft.graph.callRecords" xmlns="http://docs.oasis-open.org/odata/ns/edm" xmlns:ags="http://aggregator.microsoft.com/internal" xmlns:odata="http://schemas.microsoft.com/oDataCapabilities">
      <EntityType Name="callRecord" ags:IsOwner="true" ags:AddressUrl="https://plat.teams.microsoft.com">
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
