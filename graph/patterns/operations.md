# Operations

Microsoft Graph API Design Pattern

*The operations pattern provides the ability to model a change that impacts multiple resources and can't be effectively modeled by using HTTP methods.*

## Problem

Sometimes when modeling a complex business domain, API designers need to model a business operation that effects multiple resources and needs to be performed as a single unit. Modeling the operation via HTTP methods on each individual resource might be either ineffective or not reflect how it's processed by the backend service. In addition, the operation might produce observable side effects.

## Solution

To address these use cases, API designers might use operational resources such as functions or actions.
If the operation doesn't have any side effects and MUST return a single instance of a type or a collection of instances, then the designer SHOULD use the OData function; otherwise, the designer can model the operation as an action.

## When to use this pattern

The operations pattern is well suited to use cases that cannot be modeled as a single HTTP method on a resource and require either multiple round trips to complete a single logical operation or produce one or multiple side effects.

You can consider related patterns such as [long running operations](./long-running-operations) and [change tracking](./change-tracking).

## Issues and considerations

- Microsoft Graph does NOT support unbound actions or functions. Bound actions and functions are invoked on resources matching the type of the binding parameter. The binding parameter can be of any type, and it MAY be Nullable. For Microsoft Graph, actions and functions must have the `isBound="true"` attribute. The first parameter is the binding parameter.

- Both actions and functions support overloading, meaning a schema might contain multiple actions or functions with the same name. The overload rules as per the OData [standard](http://docs.oasis-open.org/odata/odata-csdl-xml/v4.01/odata-csdl-xml-v4.01.html#sec_FunctionOverloads) apply when adding parameters to actions and functions.
  
- Because Microsoft Graph only supports bound actions and functions, all must have at least one parameter where the first is the binding parameter. The MUSTS of parameters are as follows:

  - Each parameter must have a simple identifier name.
  - The parameter name must be unique within the overload.
  - The parameter must specify a type.

- Microsoft Graph supports the use of optional parameters. The optional parameter annotation can be used instead of creating function or action overloads when unnecessary.

- API designer **MUST** use POST to call operations on resources.

- The addition of a new mandatory not-nullable parameter to an existing action or function is a breaking change and is not allowed without proper versioning.

## Example

```
 <Action Name="createUploadSession" IsBound="true" ags:EnabledForPassthrough="true" ags:OwnerService="Microsoft.Exchange">
        <Parameter Name="bindingParameter" Type="Collection(graph.attachment)" />
        <Parameter Name="AttachmentItem" Type="graph.attachmentItem" Nullable="false" />
        <ReturnType Type="graph.uploadSession" />
      </Action>
```
 <Action Name="deprovision" IsBound="true" ags:OwnerService="Microsoft.Intune.Devices">
        <Parameter Name="bindingParameter" Type="graph.managedDevice" />
        <Parameter Name="deprovisionReason" Type="Edm.String" Nullable="false" Unicode="false" />
</Action>

 <Function Name="additionalAccess" IsBound="true" ags:OwnerService="Microsoft.IGAELM">
        <Parameter Name="bindingParameter" Type="Collection(graph.accessPackageAssignment)" />
        <ReturnType Type="Collection(graph.accessPackageAssignment)" />
 </Function>

  <Function Name="compare" IsBound="true" ags:OwnerService="Microsoft.Intune.DeviceIntent">
        <Parameter Name="bindingParameter" Type="graph.deviceManagementIntent" />
        <Parameter Name="templateId" Type="Edm.String" Unicode="false" />
        <ReturnType Type="Collection(graph.deviceManagementSettingComparison)" />
 </Function>

