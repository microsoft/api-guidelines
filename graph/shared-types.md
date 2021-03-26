[[_TOC_]]

# Overview

This topic describes three shared types scenarios:

1. Shared type object reuse (enums and complex types)
2. Extending existing resources (entity types) that another workload masters
3. Shared entity types - where different workloads want to share a common schema

For each scenario we'll show you how to update your workload schema (with the requisite AGS annotated schema), with some examples.

## Shared object reuse (enums and complex types)

We encourage API owners to reuse types where possible - some examples of some common shared types being `keyValuePair` and `patternedRecurrence`.  

You can reuse any existing enum or complex type in your workload CSDL by declaring the full type definition in an identical manner to anywhere else it is declared.  Any difference in the type's public declaration across workload CSDLs that define that same type will lead to (GMM) rule validation errors. NOTE: this does not include
workload specific annotations (such as ags:WorkloadName). If you want to change anything about these types (like change a complex type to be an open type or making a property nullable), you'll need to make that change in **all** workload CSDLs that declare that type, and then get sign-off from those workload owners.

## Referencing and extending existing schema (entity types)

You will frequently need to connect your models/APIs to other entities in Microsoft Graph. This is a key value of Microsoft Graph. For example, you might have some data or properties that you want to add and surface on another entity type that is mastered by another workload.  An example of this might be to add a `signInActivity` (complex type) property to the existing `user` entity type.  The data for `signInActivity` comes from **workload A**, while `user` is mastered by **DirectoryServices workload**. This "extending existing schema" is sometimes also referred to as a "composite type" where querying the entity returns data from multiple workloads (although we strive to hide that fact from the Microsoft Graph caller).

> **NOTE**:  You cannot extend [shared entity types](#shared-entity-types).

### How to

To enable this, you will have to provide reference versions of the entities you need in your own workload schema.

- Add a declaration to your own workload schema for the entity type(s) you want to extend. This will live under your own namespace, so refer to it as though it was in your own namespace.
- Omit the `ags:IsMaster` annotation to indicate that your service does not master this entity type
- Do not express inheritance relationships between external entities
- Do not include a copy of the 'virtual' entity named `entity` - this is synthesized by Microsoft Graph itself
- Add your `ags:AddressUrl` for the endpoint that implements the entity type in your service. It should be added to an "entry point" in your schema. In this case, that's either on a singleton or entity set. In general, other entry points could be navigation properties, functions or actions.
- Extend the entity type with the set of properties that your service masters.  These can be primitive type properties, complex type properties, navigation properties or even binding functions and actions.

### Examples

Here's an example for `signInActivity` taken from the Microsoft.AAD.Reporting.csdl workload, that adds a property to the `user` entity type. This property will show up in the user entity type definition in the [Microsoft Graph CSDL](https://graph.microsoft.com/beta/$metadata). For the sake of brevity the complex type definition for `signInActivity` is not shown:

```xml
  <!--Defined for referencing and extending user entity type -->
  <EntityType Name="user">
    <Property Name="signInActivity" Type="self.signInActivity" />
  </EntityType>

  <EntityContainer Name="AADAuditActivities">
    <EntitySet Name="users" EntityType="self.user" ags:AddressUrl="https://reportingservice.activedirectory.windowsazure.com">
  </EntityContainer>
```

Here's another (abbreviated) example from the Microsoft.Exchange.csdl workload where a `messages` contained navigation property is added to the `user` entity type, to enable paths such as `https://graph.microsoft.com/v1.0/users/{id}/messages`.

```xml
  <EntityType Name="user" BaseType="Microsoft.OutlookServices.directoryObject" ags:WorkloadName="User">
    <NavigationProperty Name="messages" Type="Collection(Microsoft.OutlookServices.message)" ContainsTarget="true" ags:WorkloadName="Messages">
  </EntityType>
  <EntityContainer Name="OutlookServices">
    <EntitySet Name="users" EntityType="Microsoft.OutlookServices.user" ags:AddressUrl="https://outlook.office365.com/api/beta/Users('[IdorUpn]')/" ags:AddressUrlMSA="https://outlook.office365.com/api/beta/Users('[sourceEntityId]')/" ags:PassthroughAddressUrl="https://outlook.office365.com/api/gbeta/Users('[IdorUpn]')/" ags:PassthroughAddressUrlMSA="https://outlook.office365.com/api/gbeta/Users('[sourceEntityId]')/" >
  </EntityContainer>
```

### Paging and filtering

When extending existing entity types (with standard properties), for those properties to be returned, the caller **must** use `$select` (i.e. they do not return by default). The Microsoft Graph front-end (AGS) will send requests to all endpoints that contribute to the "composite type" and merge the result.

To enable **paging** (and filtering), your service will need to implement **bulk fetch operation** support, typically in the form of an action that gets records by IDs, similar to what [Directory Services offers](https://docs.microsoft.com/graph/api/directoryobject-getbyids?view=graph-rest-1.0&tabs=http). The request body that AGS sends will contain an "ids" array property. This does not need to be a public action and can remain internal.  Once you have this bulk fetch by IDs mechanism, you need to register it using workload service configuration:

```xml
  <ServiceProperties>
    <Items>
      <KeyValuePairItem Key="BulkFetchUri" Value="https://workloadUriToBulkFetchItemsViaPost.com/somePathToGetByIds" />
    </Items>
  </ServiceProperties>
```

You can see an example of this in [Microsoft.AAD.Reporting service config](https://microsoftgraph.visualstudio.com/onboarding/_git/AGS-OnboardingAutomationPipeline?path=%2FMicrosoft.AAD.Reporting_WorkloadConfig.config&version=GBconfig%2Fprd%2FringSMK&_a=contents).

#### How this works

> **NOTE**: Simply adding the BulkFetchUri config and implementing the bulk fetch operation lights up the following paging and filtering support.  No further work is necessary.

For a _standard paging request_ using `$select` - for example `GET https://graph.microsoft.com/beta/users?$select=id,displayName,signInActivity`- the first page of results is fetched from the workload that masters the entity type. The IDs from the response are used in the bulk fetch request to any other workloads that master properties in the $select. The results are stitched together and the page of merged results are returned to the caller (with a standard nextlink to the next page of results).  You can see how this works using the `$whatif` query parameter - just append this to the GET request above, in [Graph Explorer](https://aka.ms/ge):

```json
{
    "Description": "Foreach entity obtained from Request1, run Request2 and merge responses:",
    "Request1": {
        "Description": "Execute HTTP request",
        "Uri": "https://graph.windows.net/v2/72f988bf-86f1-41af-91ab-2d7cd011db47/users?$select=id,displayName,id",
        "HttpMethod": "GET",
        "TargetWorkloadId": "Microsoft.DirectoryServices"
    },
    "Request2": {
        "Description": "Execute HTTP request",
        "Uri": "https://reportingservice.activedirectory.windowsazure.com/auditLogs/userSignInActivity/Default.GetByIds?$select=signInActivity",
        "HttpMethod": "GET",
        "TargetWorkloadId": "Microsoft.AAD.Reporting"
    }
}
```

If you want to offer _paging **and** filtering_, this is also possible - with a [limitation](#current-limitations). The order of operation is the same if filtering on properties from Directory Services (like `displayName`, `department` etc).  However, if filtering on a property mastered by a different workload, the request goes there first. So for example, adding a filter clause on `signInActivity` and using our `$whatif` query parameter yields:

```json
{
    "Description": "Foreach entity obtained from Request1, run Request2 and merge responses:",
    "Request1": {
        "Description": "Execute HTTP request",
        "Uri": "https://reportingservice.activedirectory.windowsazure.com/users?$filter=signInActivity%2flastSignInDateTime+le+2020-06-01T00%3a00%3a00Z&$select=signInActivity,id",
        "HttpMethod": "GET",
        "TargetWorkloadId": "Microsoft.AAD.Reporting"
    },
    "Request2": {
        "Description": "Execute HTTP request",
        "Uri": "https://graph.windows.net//v2/72f988bf-86f1-41af-91ab-2d7cd011db47/getObjectsById?$select=id,displayName",
        "HttpMethod": "GET",
        "TargetWorkloadId": "Microsoft.DirectoryServices"
    }
}
```

Here you can see that the order of operation has changed, and in Request2 it's DirectoryServices that's called with it's bulk fetch mechanism.  

> **IMPORTANT**: To support filtering for properties mastered outside of the workload that masters the entity type, all workloads (including the master workload) that contribute to the "composite type" **must** support and be configured with a bulk fetch operation.

### Current limitations

1. Filtering on properties from multiple workloads is not supported.
2. Fan-out writes are not supported for composite types:
   - POST to the entity type with properties defined in multiple workloads is not supported.  Callers need to create the entity first (with properties from the master workload only), followed by a PATCH to add the other properties. **NOTE**: there may be delays before the PATCH operation will work (due to replication/sync delays).
   - Similarly PATCH containing properties mastered by multiple workloads is also not supported.

The fan-out write scenarios may be better supported using the [instant-on mechanism](../../../Service/Proxy/Instant-On).

## Shared entity types

For this scenario, API owners in different teams/workloads want to share a common set of schema, or entity types. This provides Microsoft Graph developers with consistent objects and experiences for functionality (that unbeknown to them) is spread across multiple services.  Examples candidates for this scenario:

* Common audit event entity type (or base type)
* M365 unified RBAC APIs has a common entity types for role definitions and role assignments, used by multiple teams (Azure AD, Intune and Exchange).

### How to

To implement shared types is pretty simple:

* One workload CSDL defines the entity type (and all of its properties)
* The other workloads
   * define the same entity type, but do not need to define its properties. This **is** different from shared complex types.
   * must **not** define any additional properties in the type definition. You cannot extend a shared entity type. This would be counter to shared entity types being the same.
* All workloads (including the one that defines the type) mark the shared entity type as shareable - using the `ags:IsSharedEntity="true"` AGS annotation.
* Do **not** mark any of the shared entity types with `ags:IsMaster="true"` AGS annotation.

The shared types can be used in their own workload like any other entity type:

* in navigation properties
* have functions or actions bound to them
* used as a base class for derived types 

### Example

We'll use the M365 unified RBAC APIs as an example.
Here the Enterprise RBAC workload defines the unified role definition and unified role assignment entity types together with its properties. Both Directory Services and Intune workload also define the same types but without its properties.

**In Microsoft.EnterpriseRbac.csdl**:

This workload **fully** defines the `unifiedRoleDefinition` and the `unifiedRoleAssignment` entity types with the sharing annotation. 

```xml
  <EntityType ags:AddressUrl="https://graph.windows.net/v2/" Name="unifiedRoleDefinition" ags:WorkloadName="RoleDefinition" ags:AddressContainsEntitySetSegment="true" ags:IsSharedEntity="true">
    <Key>
      <PropertyRef Name="id" />
    </Key>
    <Property Name="id" Type="Edm.String" Nullable="false" />
    <Property Name="description" Type="Edm.String" />
    <Property Name="displayName" Type="Edm.String" />
    <Property Name="isBuiltIn" Type="Edm.Boolean" />
    <Property Name="isEnabled" Type="Edm.Boolean" />
    <Property Name="resourceScopes" Type="Collection(Edm.String)" Nullable="false" />
    <Property Name="rolePermissions" Type="Collection(Microsoft.EnterpriseRbac.unifiedRolePermission)" Nullable="false" />
    <Property Name="templateId" Type="Edm.String" />
    <Property Name="version" Type="Edm.String" />
  </EntityType>
  <EntityType ags:AddressUrl="https://graph.windows.net/v2/" Name="unifiedRoleAssignment" ags:WorkloadName="RoleAssignment" ags:AddressContainsEntitySetSegment="true" ags:IsSharedEntity="true">
    <Key>
      <PropertyRef Name="id" />
    </Key>
    <Property Name="id" Type="Edm.String" Nullable="false" />
    <Property Name="roleDefinitionId" Type="Edm.String" Nullable="true"/>
    <NavigationProperty Name="roleDefinition" Type="Microsoft.EnterpriseRbac.unifiedRoleDefinition" Nullable="true"/>
    <Property Name="condition" Type="Edm.String" Nullable="true" />
    <Property Name="principalId" Type="Edm.String" Nullable="true" />
    <NavigationProperty Name="principal" Type="Microsoft.EnterpriseRbac.DirectoryObject" Nullable="true" />
    <Property Name="directoryScopeId" Type="Edm.String" Nullable="true" />
    <NavigationProperty Name="directoryScope" Type="Microsoft.EnterpriseRbac.DirectoryObject" Nullable="true" />
    <Property Name="appScopeId" Type="Edm.String" Nullable="true" />
    <NavigationProperty Name="appScope" Type="Microsoft.EnterpriseRbac.appScope" ContainsTarget="true" Nullable="true" />
    <Property Name="resourceScope" Type="Edm.String" Nullable="true" />
  </EntityType>
```

**In Microsoft.DirectoryServices.csdl**:

This workload **only** defines the `unifiedRoleDefinition` and the `unifiedRoleAssignment` entity types without any properties, but with the sharing annotation. 

```xml
    <EntityType ags:IsMaster="false" ags:AddressUrl="https://graph.windows.net/v2/" Name="unifiedRoleDefinition" ags:WorkloadName="RoleDefinition" ags:AddressContainsEntitySetSegment="true" ags:IsSharedEntity="true">
    </EntityType>

    <EntityType ags:IsMaster="false" ags:AddressUrl="https://graph.windows.net/v2/" Name="unifiedRoleAssignment" ags:WorkloadName="RoleAssignment" ags:AddressContainsEntitySetSegment="true" ags:IsSharedEntity="true">
    </EntityType>
```
