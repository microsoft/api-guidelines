### Deprecation Guidelines

If your API requires the introduction of breaking changes you must add Revisions annotations to the API definition with the following terms:

    
    -   Date: Date when the element was marked as deprecated.
    -   Version: Used to organize the ChangeLog. Use the format "YYYY-MM/Category" where "YYYY-MM" is the month the deprecation is announced, and "Category" is the category under which the change is described.
    -   Kind: Deprecated - 
    -   Description: Human readable description of the change: Used in changelog, documentation etc.
    -   RemovalDate: Earliest date when the element may be removed.

The annotation can be applied to a type, an entity set, a singleton,a property, a
navigation property, a function or an action. If a type is marked as deprecated, it
is not necessary to mark the members of that type as deprecated, nor is it necessary
to annotate any usages of that type.

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

When the request URL contains a reference to a deprecated model element, the gateway will add a [Deprecation
header](https://tools.ietf.org/html/draft-dalal-deprecation-header-02) (with the
date the element was marked as deprecated) and a Sunset header (with the date 2
years beyond the Deprecation date) to the response.

**Deprecation header example:**

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Deprecation: Thursday, 30 June 2022 11:59:59 GMT
Sunset: Wed, 30 Mar 2022 23:59:59 GMT
Link: https://docs.microsoft.com/en-us/graph/changelog#2022-03-30_name ; rel="deprecation"; type="text/html"; title="name",https://docs.microsoft.com/en-us/graph/changelog#2020-06-30_state ; rel="deprecation"; type="text/html"; title="state"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
