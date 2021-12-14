### Deprecation Process

If your API requires an introduction of breaking changes you must follow the
deprecation process:

-   After API review board approvals, add Revisions annotation to the API
    definition CSDL with the following terms:

    -   Kind of change: Deprecated (vs "added" to track added properties/types)

    -   Human readable description of the change: Used in changelog,
        documentation etc.

    -   Version: Used to identify group of changes. Of the format
        "YYYY-MM/Category" where "YYYY-MM" is the month the deprecation is
        announced, and "Category" is the category under which the change is
        described in the ChangeLog

    -   Date: Date when the element was marked as deprecated

    -   RemovalDate: Date when the element may be removed

The annotation can be applied to a type, entity set, singleton, property,
navigation property, function or action. If a type is marked as deprecated, it
is not necessary to mark members of that type as deprecated, nor is it necessary
to annotate any usage of that type in entity sets, singletons, properties,
navigation properties, functions, or actions.

**Example of property annotation:**

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 <EntityType Name="outlookTask" BaseType="Microsoft.OutlookServices.outlookItem" ags:IsMaster="true" ags:WorkloadName="Task" ags:EnabledForPassthrough="true">
    <Annotation Term="Org.OData.Core.V1.Revisions">
      <Collection>
        <Record>
          <PropertyValue Property = "Date" Date="2020-08-20"/>
          <PropertyValue Property = "Version" String="2020-08/Tasks_And_Plans"/>
          <PropertyValue Property = "Kind" EnumMember="Org.OData.Core.V1.RevisionKind/Deprecated"/>
          <PropertyValue Property = "Description" String="The Outlook tasks API is deprecated and will stop returning data on August 20, 2022. Please use the new To Do API."/>
          <PropertyValue Property = "RemovalDate" Date="2022-08-20"/>
        </Record>
      </Collection>
    </Annotation>
    ...
  </EntityType>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When the request URL contains a reference to a deprecated model element, the
HTTP response includes a [Deprecation
header](https://tools.ietf.org/html/draft-dalal-deprecation-header-02) (with the
date the element was marked as deprecated) and a Sunset header (with the date 2
years beyond the Deprecation date). Response also includes a link header
pointing to the breaking changes page.

**Deprecation header example:**

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Deprecation: Thursday, 30 June 2022 11:59:59 GMT
Sunset: Wed, 30 Mar 2022 23:59:59 GMT
Link: https://docs.microsoft.com/en-us/graph/changelog#2022-03-30_name ; rel="deprecation"; type="text/html"; title="name",https://docs.microsoft.com/en-us/graph/changelog#2020-06-30_state ; rel="deprecation"; type="text/html"; title="state"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Deprecation sequence:**

-   As an API developer you can mark individual API schema elements as
    deprecated 
    TODO
     deprecation cadence will allow the services to
    evolve schemas over time, without waiting for a coordinated, monolithic
    endpoint change.

-   Once marked as deprecated, the elements must continue to be supported for a
    minimum of 3 years before removal (or a minimum of 2 years if, based on
    telemetry, the element is no longer being used).

-   Tools, documentation, SDKs, and other mechanisms are driven by this explicit
    deprecation to reach out to customers that may be affected by the changes.

-   APIs in beta or preview versions can use the same mechanism but are not
    bound by the quarterly cadence or minimal support period before removal of
    deprecated elements.