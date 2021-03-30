# Type namespaces

Types should be declared in an appropriate namespace. As with traditional compiled libraries, putting types in namespaces creates a better developer experience.

> Warning<br/>
> Type namespaces are not 1:1 with URL segmentation. For guidance on URL segmentation, see [[Singletons]].

## Namespace usage

Namespaces are used both at runtime and in the developer experience.

### Runtime

At runtime, namespaces appear when using type cast segments and in `@odata.type` annotations.

Type cast segments allow the caller to filter a collection to a given subtype. This usage is rare, but does happen in Microsoft Graph, e.g.:

```http
GET https://graph.microsoft.com/v1.0/directory/deletedItems/microsoft.graph.user
```

We are working to make type cast segment unqualified when the type name is unambiguous. The effect of this change will be that the namespace in the type cast segment is only required when multiple types with that name exist.

```http
GET https://graph.microsoft.com/v1.0/directory/deletedItems/user
```

Type cast segments also appear in `@odata.type` annotations. `@odata.type` annotations are only included if:

- The type is ambiguous, such as when the type appears in a collection of a supertype.
- Type information was explicitly requested by asking for `odata.metadata=full`.

### Developer experience

Namespaces also affect the developer experience:

| Area           | Effect                                            |
| -------------- | ------------------------------------------------- |
| SDKs           | Types are generated in the appropriate namespace. |
| Docs           | API reference is organized by namespace.          |
| Changelog      | Changes are categorized by namespace.             |
| Graph Explorer | No effect.                                        |

## Namespace declaration

The type namespace is declared in the `Namespace` attribute on the `Schema` that is uploaded to Microsoft Graph.

```xml
<Schema Namespace="microsoft.graph.{namespace}">
...
</Schema>
```

### Historic behavior

Until late 2018, Microsoft Graph merged all types into a single namespace, `microsoft.graph`, effectively ignoring the namespace specified in the schema. When the same type name appeared in multiple schemas, the types were merged. This provides teams with the ability to extend a type owned by a different team.

There are several facets of the historic behavior that should be maintained:

- Workloads should be able to explicitly declare types in the namespace `microsoft.graph`.
- Workloads should be able to extend types owned by other workloads.
- Types with the same name and namespace should be merged to allow extensibility.

### Backwards compatibility

It is a breaking change to change a type's namespace. Types that are in the namespace `microsoft.graph` in `v1.0` must stay in that namespace until Graph 2.0. A workload may technically start introducing types in their custom namespace even if the bulk of their types are in `microsoft.graph`. It is up to API reviewers to determine on a case-by-case basis whether this makes sense.

## Namespace heuristic

Microsoft Graph uses a specific heuristic to determine whether the namespace specified on the schema should be exposed publicly. This has two advantages.

1. No changes need to be made to existing schemas or workloads to ensure that they keep working without introducing a breaking change.
2. The heuristic enforces that all namespaces must begin with `microsoft.graph.`.

The heuristic is simple: if the namespace string starts with `microsoft.graph.` (case-insensitive, but must include the trailing `.`), the namespace will be publicly exposed. If the namespace does not begin with that exact string, all types in the schema will be coerced into the `microsoft.graph` namespace.

| Schema namespace (case-insensitive match)    | Public namespace                             |
| -------------------------------------------- | -------------------------------------------- |
| `MyNamespace`                                | `microsoft.graph`                            |
| `Microsoft.Graph`                            | `microsoft.graph`                            |
| `Microsoft.Graph.MyNamespace`                | `microsoft.graph.myNamespace`                |
| `Microsoft.Graph.MyNamespace.MySubNamespace` | `microsoft.graph.myNamespace.mySubNamespace` |

### Namespace coercion

Namespaces will be coerced in two ways:

1. Casing will be coerced to `lowerCamelCase`.
2. Namespace length will be coerced to 4 segments.

| Schema namespace (case-insensitive match) | Coerced namespace                  |
| ----------------------------------------- | ---------------------------------- |
| `microsoft.graph.MyNamespace`             | `microsoft.graph.myNamespace`      |
| `microsoft.graph.myNamespace.sub1.sub2`   | `microsoft.graph.myNamespace.sub1` |

## Namespace ownership

Graph does not enforce namespace ownership. However, namespaces do have key contacts that should be consulted when modifying types in that namespace.

Types must exist within a namespace, and workloads must explicitly state which namespace a type exists in. This does not mean that a workload must have their own namespace. A workload could continue to use their internal namespace or explicitly state that the types are in `microsoft.graph`.

| Namespace                   | Owner                                                            |
| --------------------------- | ---------------------------------------------------------------- |
| microsoft.graph.callRecords | [IC3 Records Distribution team](mailto:ic3recdist@microsoft.com) |

## Extensibility and cross-referencing

To extend a type in a different schema, a workload must declare that schema and the type in it. This is conceptually similar to .NET partial types.

To reference a type in a different schema, simply refer to that type by fully qualified name (namespace + type name).

OData fully supports cross-referencing and will not fail even if a cycle happens between schemas. That said, normal cyclical constraints apply - a cycle in inheritance will break things.

## Managing multiple schemas

Workloads must define schemas in their csdl using the Edmx format. [Microsoft.IC3.DataPlatform](https://microsoftgraph.visualstudio.com/onboarding/_git/AGS-OnboardingAutomationPipeline?path=%2FMicrosoft.IC3.DataPlatform.csdl&version=GBschemas%2Fprd%2Fbeta&_a=contents) is an example of a workload that exposes multiple namespaces.

::: tip
As with schemas that exist in the `microsoft.graph` namespace, defining an `entity` type is optional, AGS will transform your schema to make all entity types derive from `microsoft.graph.entity`.
:::

::: warning
Do not deviate from the general structure in the example below. GMM expects the XML structure (including xml namespace declarations) to match the example below.
:::

```xml
<?xml version="1.0" encoding="utf-8"?>
<edmx:Edmx Version="4.0" xmlns:edmx="http://docs.oasis-open.org/odata/ns/edmx" xmlns:ags="http://aggregator.microsoft.com/internal" xmlns:odata="http://schemas.microsoft.com/oDataCapabilities">
  <edmx:DataServices>
    <Schema Namespace="microsoft.graph.callRecords" xmlns="http://docs.oasis-open.org/odata/ns/edm" xmlns:ags="http://aggregator.microsoft.com/internal" xmlns:odata="http://schemas.microsoft.com/oDataCapabilities">
      <EntityType Name="callRecord" ags:IsMaster="true" ags:AddressUrl="https://plat.teams.microsoft.com">
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
        <NavigationProperty Name="callRecords" Type="Collection(microsoft.graph.callRecords.callRecord)" ContainsTarget="true" ags:AddressUrl="https://plat.teams.microsoft.com/communications/callRecords/[sourceEntityId:0]" ags:AddressUrlMSA="https://plat.teams.microsoft.com/communications/callRecords/[sourceEntityId:0]" ags:AddressContainsEntitySetSegment="true" />
      </EntityType>
    </Schema>
  </edmx:DataServices>
</edmx:Edmx>
```

### Public `$metadata`

The publicly hosted `$metadata` endpoint will have multiple schemas - one per coerced namespace. The primary entity container will exist in the schema with the `microsoft.graph` namespace.

```xml
<edmx:Edmx xmlns:edmx="http://docs.oasis-open.org/odata/ns/edmx" Version="4.0">
  <edmx:DataServices>
    <Schema xmlns="http://docs.oasis-open.org/odata/ns/edm" Namespace="microsoft.graph">
      <EntityType Name="entity" Abstract="true">
        <Key>
          <PropertyRef Name="id"/>
        </Key>
        <Property Name="id" Type="Edm.String" Nullable="false"/>
      </EntityType>
      <EntityContainer Name="GraphService">
        <EntitySet Name="messages" EntityType="microsoft.graph.outlook.mail.message"/>
      </EntityContainer>
    </Schema>
    <Schema xmlns="http://docs.oasis-open.org/odata/ns/edm" Namespace="microsoft.graph.outlook">
      <EntityType Name="outlookItem" BaseType="microsoft.graph.entity" Abstract="true">
        ...
      </EntityType>
    </Schema>
    <Schema xmlns="http://docs.oasis-open.org/odata/ns/edm" Namespace="microsoft.graph.outlook.mail">
      <EntityType Name="message" BaseType="microsoft.graph.outlook.outlookItem" OpenType="true">
        ...
      </EntityType>
    </Schema>
  </edmx:DataServices>
</edmx:Edmx>
```
