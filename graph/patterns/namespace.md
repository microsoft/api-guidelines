# Namespace

Microsoft Graph API Design Pattern

### *The Namespace provides the ability to group resource definitions together into a logical set.*

## Problem

When building a complex offering API designers may need to model many different
resources and their relationships. For better user experience and
discoverability related API definitions need to be grouped together. Large API
surface may need to be partitioned to generate light-weight client SDKs and libraries.

## Solution

API designers can use the Namespace attribute of the CSDL schema to
declare a namespace and logically bundle related API entities in the Graph
metadata.
```XML
<Schema Namespace="microsoft.graph.{namespace}">
...
</Schema>
```

All resources declared within the namespace will be grouped together in the public
Graph metadata and may be used for partitioning.

A public namespace must have "microsoft.graph.” prefix and be presented in camel
case, i.e microsoft.graph.myNamespace.

When type casting is required in the API query, request or response a fully-qualified type name is represented as concatenation of a namespace and a type name. Consequently namespace should be aligned with API category 
path segment.

## When to Use this Pattern

API resource grouping creates a user-friendly experience keeping all resources
for a specific use case close together. It also allows generating smaller SDKs
and libraries and limits length of IDE prompts such as Intellisense.


## Issues and Considerations

1.  Microsoft Graph consistency requirements discourage using the same type
    names for different concepts even within different namespaces. Microsoft
    Graph type names must be descriptive and almost always unique within the API
    surface.

2.  Namespace must be consistent with API navigation path.

3.  When type name is ambiguous and requires a namespace qualifier changing namespace is a breaking change.
    

4.  To extend a type in a different schema, a service must declare that schema
    and the type in it. This is conceptually similar to .NET partial types.

5.  To reference a type in a different schema, simply refer to that type by
    fully qualified name (namespace + type name).
6. Microsoft Graph has heuristic rules for declared namespaces:

   -   All public namespaces must have a prefix ‘microsoft.graph’

   -   If a namespace does not begin with ‘microsoft.graph’ prefix, all types in
       the schema will be coerced into the main ‘microsoft.graph’ namespace.

## Example

Namespace and type declarations:
```XML

<Schema Namespace="microsoft.graph.search" xmlns=”<http://docs.oasis-open.org/odata/ns/edm>”\>
…
    <EntityType Name="bookmark" …
    </EntityType>
…
</Schema>

```

Fully qualified type name: microsoft.graph.search.bookmark
