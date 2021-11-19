# Microsoft Graph REST API Guidelines

Table of Contents

- [Microsoft Graph REST API Guidelines](#microsoft-graph-rest-api-guidelines)
  - [](#)
      - [History](#history)
  - [Introduction](#introduction)
    - [Legend](#legend)
  - [Design Approach](#design-approach)
    - [Naming](#naming)
    - [Uniform Resource Locators (URLs)](#uniform-resource-locators-urls)
    - [Query Support](#query-support)
    - [Recommended Modeling Patterns](#recommended-modeling-patterns)
  - [Behavior Modeling](#behavior-modeling)
      - [Microsoft Graph rules for modeling behavior](#microsoft-graph-rules-for-modeling-behavior)
    - [Error Handling](#error-handling)
    - [API contract and non-backward compatible changes](#api-contract-and-non-backward-compatible-changes)
  - [Versioning and Deprecation](#versioning-and-deprecation)
    - [Deprecation Process](#deprecation-process)
  - [Recommended API Patterns](#recommended-api-patterns)
  - [References](#references)

## 

#### History

| Date        | Notes                       |
|-------------|-----------------------------|
| 2021-Sep-28 | Alignment with Azure style. |
| 2020-Oct-04 | Initial version in Wiki.    |

## Introduction

When building a digital ecosystem you should use API-first approach and start
with design and development of your APIs. Considering API usability and creating
APIs that are easy to discover, simple to use, fit to purpose, and consistent
across your products will make the difference between success and failure of
your ecosystem.

This document offers guidance that Graph API developer teams MUST follow to
ensure that customers have a great experience. A new API design should meet the
following goals:

\- Developer friendly via consistent naming, patterns, and web standards (HTTP,
REST, JSON)

\- Efficient and cost-effective.

\- Work well with SDKs in many programming languages.

\- Sustainable & versionable via clear API contracts.

The Microsoft Graph guidelines are an extension of the Microsoft REST API
guidelines. Readers are assumed also be reading the Microsoft REST API
guidelines and be familiar with them. Graph guidance is a superset of the
Microsoft API guidelines and services should follow them except where this
document outlines specific differences or exceptions to those guidelines.
Together these guidelines and a library of API patterns serve as the means by
which API teams discuss and come to consensus on API review recommendations.

This document borrows heavily from multiple public sources such as:

1.  Microsoft Azure REST API Guidelines

2.  Google Cloud Platform APIs

3.  WSO2 Rest API Design Guidelines and others.

Technology and software are constantly changing and evolving, and as such, this
is intended to be a living document. [Open an
issue](https://github.com/microsoft/api-guidelines/issues/new/choose) to suggest
a change or propose a new idea.

### Legend

This document offers prescriptive guidance labeled as follows:

:heavy_check_mark: **DO** satisfy this specification. If not following this
advice, you MUST disclose your reason during the Graph API review.

:no_entry: **DO NOT** use this pattern. If not following this advice, you MUST
disclose your reason during the Graph API review.

:ballot_box_with_check: **YOU SHOULD** fulfill this specification. If not
following this advice, you MUST disclose your reason during the Graph API
review.

## Design Approach

The design of your API is arguably the most important investment you will make
in it. The design of your API is what creates the first impression for
developers. Microsoft Graph APIs follow HTTP, REST, and JSON standards and are
described using ODATA conventions and CSDL for schema definition (see
[Documentation · OData - the Best Way to
REST](https://www.odata.org/documentation/)).

We promote API-first design approach where you begin by creating an interface or
API for your service first. Subsequently you follow with the service
implementation which relies on the specified interface. API -first approach is
essential for agility, predictability, and reuse of your APIs as it promotes
good understanding of your modeling domain, consistent interface contract, and
understanding of how supporting service will evolve.

In general API design includes the following steps:

-   Define your domain model

-   Derive and name your API resources

-   Determine required behavior

-   Determine user roles and permissions

-   Specify errors

To create a good API you need to start with understanding your **use cases** and
supporting domain model. We describe domain models in terms of entities or
resources, their properties, and relationships and further refer to it as entity
data model. There is no one-to-one correspondence between domain model elements
and API resources as APIs usually support only customer-facing use cases.

After API resources are identified you need to name them and their properties so
that the API will be discoverable and intuitive for developers, and consistent
with other Graph resources.

When resources are defined it’s time to think about the behavior of your API and
define required operations and actions. There are read-only and write scenarios
where a resource can be used to represent some kind of data processing
operation. The terms function and action are used to identify read and write
operation style resources, respectively.

At every step of your design you need to consider security, privacy and
compliance as an intrinsic components of your API implementation. And finally
based on your API resources, their behavior, and anticipated exceptions you need
to identify potential error scenarios with secure and descriptive messaging.

### Naming

Consistent naming is foundational for API usability. API resources are typically
described by nouns. You need to consider that resources and property names
appear in API URLs and payloads and should be descriptive and easy to
understand. Microsoft Graph naming conventions follow [Microsoft REST API
Guidelines](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md).
Below is a short summary of the most often used conventions.

| Requirements                                                                                                         | Example                                                                                                                                                                                                                                                                                                                 |
|---------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| :no_entry: **MUST NOT** use redundant words in names.   |- **Right:** /places/{id}/**type** and /phones/{id}/**number** <BR> -  **Wrong** /places/{id}/*placeType* and /phones/{id}/**phoneNumber** |
| :no_entry: **SHOULD NOT** use brand names in type or property names.                                 | - **Right:** chat   <BR> -  **Wrong** teamsChat                                                                                              |
| :no_entry: **SHOULD NOT** use acronyms or abbreviations unless they are broadly understood.          | - **Right:** url or htmlSignature <BR> - **Wrong** msodsUrl or dlp                                                                        |
| :heavy_check_mark: **MUST** use singular nouns for type names.                                              | - **Right:** address  <BR> - **Wrong** addresses                                                                                           |
| :heavy_check_mark: **MUST** use plural nouns for collections (for listing a type or collection properties). | - **Right:** addresses <BR> - **Wrong** address                                                                                           |
| :heavy_check_mark: **SHOULD** pluralize the noun even when followed by an adjective (a "postpositive").       | - **Right:** passersby or mothersInLaw    <BR> -  **Wrong** notaryPublics or motherInLaws                                                     |
| **casing** | |
| :heavy_check_mark: **MUST** use lower camel case for *all* names and namespaces                                                                                                                                                                                                      | - **Right:** automaticRepliesStatus. <BR> - **Wrong** kebab-case or snake_case.                        |
| :heavy_check_mark: **SHOULD** case two-letter acronyms with the same case.                                                                                                                                                                                                           | - **Right:** ioLimit or totalIOAmount <BR> - **Wrong** iOLimit or totalIoAmount                        |
| :heavy_check_mark: **SHOULD** case three+ letter acronyms the same as a normal word.                                                                                                                                                                                                 | - **Right:** fidoKey or oauthUrl <BR> - **Wrong** webHTML                                              |
| :no_entry: **MUST NOT** capitalize the word following a [prefix](https://www.thoughtco.com/common-prefixes-in-english-1692724) or words within a [compound word](http://www.learningdifferences.com/Main%20Page/Topics/Compound%20Word%20Lists/Compound_Word_%20Lists_complete.htm). | - **Right:** subcategory, geocoordinate or crosswalk <BR> - **Wrong** metaData, semiCircle or airPlane |
| :heavy_check_mark: **MUST** capitalize within hyphenated and open (spaced) compound words.                                                                                                                                                                                           | - **Right:** fiveYearOld, daughterInLaw or postOffice <BR> - **Wrong** paperclip or fullmoon           |
| **prefixes and suffixes** | |
| :heavy_check_mark: **MUST** suffix date and time properties with                                                                                  | - **Right:** dueDate — an Edm.Date <BR> - **Right:** createdDateTime — an Edm.DateTimeOffset <BR> - **Right:** recurringMeetingTime — an Edm.TimeOfDay <BR>- **Wrong** dueOn or startTime - <BR> - **Right:** instead both above are an Edm.DateTimeOffset                                                                                 |
| :heavy_check_mark: **SHOULD** use the Duration type for durations, but if using an int, append the units.                                         | - **Right:** passwordValidityPeriod — an Edm.Duration <BR> - **Right:** passwordValidityPeriodInDays — an Edm.Int32 (NOTE use of Edm.Duration type is preferable) <BR>- **Wrong** passwordValidityPeriod — an Edm.Int32                                                                                                          |
| :no_entry: **MUST NOT** use suffix property names with primitive type names unless the type is temporal.                                          | - **Right:** isEnabled or amount <BR> - **Wrong** enabledBool                                                                                                                                                                                                                                                                |
| :heavy_check_mark: **SHOULD** prefix property names for properties concerning a different entity.                                                 | - **Right:** siteWebUrl on driveItem, or userId on auditActor <BR> - **Wrong** webUrl on contact when its the companyWebUrl                                                                                                                                                                                                  |
| :heavy_check_mark: **SHOULD** prefix Boolean properties with is, unless this leads to awkward or unnatural sounding names for Boolean properties. | - **Right:** isEnabled or isResourceAccount <BR>- **Wrong** enabled or allowResourcAccount <BR> - **Right:** allowNewTimeProposals or allowInvitesFrom — subjectively more natural than the examples below <BR> - **Wrong** isNewTimeProposalsAllowed or isInvitesFromAllowed — subjectively more awkward that the examples above |
| :no_entry: **MUST NOT** use 'collection', 'response', 'request ' suffixes .                                          |- **Right:** addresses <BR> - **Wrong** addressCollection                                                                                                                                                                                                          |

### Uniform Resource Locators (URLs)

A Uniform Resource Locator (URL) is how developers access the resources of your
API.

Navigation path to the Microsoft Graph resources generally broken into multiple
segments:

**{scheme}://{host}/{version}/{category}/[{pathSegment}][?{query}]** where

-   **scheme and host segments** are always
    [https://graph.microsoft.com](https://graph.microsoft.com/v1.0/users);

-   **version** can be V1.0 or beta;

-   **category** segment is a logical grouping of APIs into top-level
    categories;

-   **pathSegment** is the last navigation segment which can address an entity,
    collection of entities, property or operation available for an entity

-   **query string** must follow the OData standard for query representations
    and is covered in [Query](#query) section of OData specifications.

While HTTP defines no constraints on how different resources are related
together, it does encourage the use of URL path segment hierarchies to convey
relationships. In Microsoft Graph lifetime relationships between resources are
supported by the OData concepts of singletons, entitySets, entities, complex
types and navigation properties.

In Microsoft Graph a top-level API category may represent one of the following
groupings:

1.  A core *user-centric concept* of the Graph

    1.  For example: /users, /groups or /me

2.  A Microsoft *product or service offerings* covering multiple use cases

    1.  For example: /teamwork, /directory

3.  A *feature* offering covering a single use case and *shared* across multiple
    Microsoft products

    1.  For example: /search, /notifications, /subscriptions, /files

4.  *Administrative configuration* functions for specific products. (Note: this
    is not final and may be adjusted based on the survey results)

    1.  For example: /admin/exchange

5.  Internal Microsoft requirements for publishing Privileged and Hidden APIs,
    routing, and load testing

    1.  For example: /loadTestEntities

Effectively top-level categories define a perimeter for the API surface thus a
new category creation requires additional rigor and governance.

### Query Support

Microsoft Graph APIs should support basic query options in conformance with
OData specifications and [Microsoft REST API
Guidelines](https://github.com/microsoft/api-guidelines/blob/master/Guidelines.md#7102-error-condition-responses).
|Requirements|
|----------------------------------------------------------------------------------------------------|
| :heavy_check_mark: **SHOULD** support \$select on resource to enable properties projection |
| :heavy_check_mark: **SHOULD** support \$filter with eq, ne operations on properties of entities for collections| :heavy_check_mark:
| :ballot_box_with_check: **SHOULD** support pagination 4top and $count for collections |

Limitations of \$query requests made to Microsoft Graph:

-   Microsoft Graph only supports having all the query options completely in the
    request body or completely in the request url. Graph doesn't support query
    options present in both places.

-   The parameters in \$query should not span multiple workloads. Support for
    \$query right now is limited to properties belonging to the same workload.

The query options part of an OData URL can be quite long, potentially exceeding
the maximum length of URLs supported by components involved in transmitting or
processing the request. One way to avoid this is to use the POST verb instead of
GET, and pass the query options part of the URL in the request body as described
in the chapter [OData Query
Options](http://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part2-url-conventions.html#sec_PassingQueryOptionsintheRequestBody).

| Additional Microsoft Graph rules for modeling resources                                                  |
|----------------------------------------------------------------------------------------------------------|
| :heavy_check_mark: **MUST** use String type for ID      |
| :heavy_check_mark: **SHOULD** use a primary key composed of a single property and not multiple. |
| :heavy_check_mark: **MUST** use an object as the root of all JSON payloads                               |
| :heavy_check_mark: **MUST** use a value property in the root object to return a collection                |
| :heavy_check_mark: **MUST** include @odata.type annotations when the type is ambiguous                    |
| :no_entry: **SHOULD NOT** add the property id to a complex type                                              |

### Recommended Modeling Patterns

There are different approaches to design an API resource model in situations
with multiple variants of common concept. Type Hierarchy, Facets, and Flat bag
of properties are three most often used patterns in Microsoft Graph today:

-   Type hierarchy is represented by one abstract base type with a few common
    properties and one sub-type for each variant [Modelling with Subtypes
    Pattern](./Modelling%20with%20Subtypes%20Pattern.md)

-   Facets are represented by a single entity type with common properties and
    one facet property (of complex type) per variant. The facet properties only
    have a value when the object represents that variant [Modelling with Facets
    Pattern](./Modelling%20with%20Facets%20Pattern.md)

-   Flat bag of properties is represented by one entity type with all the
    potential properties plus an additional property to distinguish the
    variants, often called type. The type property describes the variant and
    also defines properties that are required/meaningful for the variant given
    by the type property. [Modelling with Flat Bag
    Pattern](./Modelling%20with%20Flat%20Bag%20Pattern.md)

The following table shows summary of main qualities for each pattern and will
help to select a pattern preferred for your use case.

| API qualities Patterns | Properties and behavior described in metadata | Suited for strongly typed languages | Simple query construction | Syntactical backward compatible |
|------------------------|-----------------------------------------------|-------------------------------------|---------------------------|---------------------------------|
| Type hierarchy         | yes                                           | yes                                 | no                        | yes                             |
| Facets                 | ok                                            | ok                                  | yes                       | yes                             |
| Flat bag               | no                                            | no                                  | yes                       | yes                             |

## Behavior Modeling

The HTTP operations dictate how your API behaves. The URL of an API, along with
its request/response bodies, establishes the overall contract that developers
have with your service. As an API provider, how you manage the overall request /
response pattern should be one of the first implementation decisions you make.

#### Microsoft Graph rules for modeling behavior

| Requirements                                                                                                    | Severity |
|-----------------------------------------------------------------------------------------------------------------|----------|
| :heavy_check_mark: **MUST** use POST to create new entities in insertable entity sets                             | Error    |
| :heavy_check_mark: **MUST** use PATCH to edit updatable resources                                                 | Error    |
| :heavy_check_mark: **MUST** use DELETE to delete deletable resources                                              | Error    |
| :heavy_check_mark: **MUST** return a Location header with the edit URL or read URL of a created resource          | Error    |
| :heavy_check_mark: **MUST** use GET …/{collection} and GET …/{collection}/{id} for listing and reading resources. | Error    |
| :heavy_check_mark: **MUST** use POST …/{collection} for creating resources.                                       | Error    |
| :heavy_check_mark: **MUST** use PATCH …/{collection}/{id} for updating resources.                                 | Error    |
| :no_entry: **SHOULD NOT** use PUT …/{collection}/{id} for updating resources.                                       | Warning  |
| :no_entry: **MUST NOT** use PATCH to replaces resources or PUT to partially update resources.                     | Error    |
| :no_entry: **SHOULD NOT** use patterns that require multiple round trips to complete a single logical action.       | Warning  |
| :ballot_box_with_check: **MAY** supporting return and omit-nulls preferences.                              | Warning  |

For a complete list of standard HTTP operations you can refer to the [Microsoft
REST API
Guidelines](https://github.com/microsoft/api-guidelines/blob/master/Guidelines.md#7102-error-condition-responses).

### Error Handling

Microsoft REST API Guidelines provide guidelines that Microsoft REST APIs should
follow when returning error condition responses. However, the structure, form
and content of the error response payloads is currently not enforced leading to
undiscoverable and inconsistent error messages. You can improve API traceability
and consistency by using recommended Graph error model:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{
"error": {
    "code": "BadRequest",
    "message": "Unsupported functionality",
    "target": "query",
    "details": [
                {
                "code": "301",
                "target": "\$search",
                "message": "\$search query option not supported"
                }
                ],
    "innererror": {
                "code": "301",
                "message": "Cannot process the request because a required field is missing.",
                "stacktrace": [...],
                }
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following examples demonstrate error modeling for common use cases:

-   **Simple error**: A workload wants to report an error with top-level details
    only. Then the error object contains the top-level error code, message and
    target (optional).

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{
  "error": {
    "code": "badRequest",
    "message": "Cannot process the request because it is malformed or incorrect.",
	"target": "Service X (Optional)"
  }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-   **Detailed error**: An API needs to provide service-specific details of the
    error via the innererror property of the error object. The code property in
    innererror is optional but highly recommended. It is intended to allow
    services to supply a specific error code to help differentiate errors that
    share the same top-level error code but reported for different reasons.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{
  "error": {
    "code": "badRequest",
    "message": "Cannot process the request because it is malformed or incorrect.",
    "innererror": {
      "code": "requiredFieldOrParameterMissing",
      "message": "Cannot process the request because a required field or parameter is missing.",
      "stacktrace": "[StackTrace]"
    }
  }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| Microsoft Graph enforces the following error rules                                                                    | Severity |
|-----------------------------------------------------------------------------------------------------------------------|----------|
| :heavy_check_mark: **MUST** return an error property with a child code property in all error responses.                 | Error    |
| :heavy_check_mark: **MUST** return a 403 Forbidden error when insufficient scopes are present on the auth token.        | Error    |
| :heavy_check_mark: **MUST** return a 429 Too many requests error when the caller has exceeded throttling limits.        | Error    |
| :ballot_box_with_check: **MAY** returning a 404 Not found error if a 403 would result in information disclosure. | Warning  |

For a complete mapping of error codes to HTTP statuses you can refer to the
[rfc7231 (ietf.org)](https://datatracker.ietf.org/doc/html/rfc7231#section-6).

### API contract and non-backward compatible changes

Microsoft Graph definition of breaking changes is based on the [Microsoft REST
API
Guidelines](https://github.com/microsoft/api-guidelines/blob/graph/Guidelines.md#123-definition-of-a-breaking-change).
In general, making changes to the API contract for existing elements is
considered breaking. Adding new elements is allowed and not considered a
breaking change.

\*\* Non-breaking changes:\*\*

-   Addition of an annotation OpenType="true" Addition of properties that are
    nullable or have a default value

-   Addition of a member to an evolvable enumeration 1. Removal, rename, or
    change to the type of an open extension

-   Removal, rename, or change to the type of an annotation \*Introduction of
    paging to existing collections

-   Changes to error codes Changes to the order of properties

-   Changes to the length or format of opaque strings, such as resource IDs

\*\* Breaking changes:\*\*

-   Changes to the URL or fundamental request/response associated with a
    resource

-   Changing semantics of resource representation

-   Removal, rename, or change to the type of a declared property

-   Removal or rename of APIs or API parameters Addition of a required request
    header

-   Addition of a EnumType members for non-extensible enumerations

-   Addition of a Nullable="false" properties to existing types

-   Addition of a Nullable="false" parameters to existing actions and functions

-   Adding attributes to existing nodes is considered breaking.

For the full list of rules you can refer to [this section of the OData V4
spec](https://docs.oasis-open.org/odata/odata/v4.0/errata02/os/complete/part1-protocol/odata-v4.0-errata02-os-part1-protocol-complete.html#_Toc406398209).

## Versioning and Deprecation

When changes are imminent you need to support explicit versioning as it's
critical that clients can count on services to be stable over time, and it's
critical that services can add features and make changes. Microsoft Graph API
follows the guidance described in the Model Versioning section of the [Microsoft
REST API
guidelines](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md#12-versioning).

The following versions of the Microsoft Graph API are currently available:

1.  API sets on the v1.0 endpoint (https://graph.microsoft.com/v1.0) are in
    general availability (GA) status.

2.  API sets on the beta endpoint (https://graph.microsoft.com/beta) are in beta
    or private preview status.

In general API breaking changes are not allowed in the GA version of Microsoft
Graph API. For beta API you can expect breaking changes and deprecation of APIs
from time to time.

As new versions of the Microsoft Graph REST APIs and Microsoft Graph SDKs are
released, earlier versions will be retired. Microsoft declares a version as
deprecated at least 24 months in advance of retiring it. Similarly, for
individual APIs that are generally available (GA), Microsoft declares an API as
deprecated at least 24 months in advance of removing it from the GA version.

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

**Deprecation cadence:**

-   As an API developer you can mark individual API schema elements as
    deprecated on a quarterly basis, after going through an API review and
    approval process. Quarterly deprecation cadence will allow the services to
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

## Recommended API Patterns

The guidelines in previous sections are intentionally brief and provide a jump
start for Graph API developers. More detailed design guidance on REST APIs is
published at the [Microsoft REST API
Guidelines](https://github.com/microsoft/api-guidelines/) and Graph specific
patterns are outlined in the table below.

Recommended API Design patterns:

| Pattern                 | Description                                                                                    | Reference                                                            |
|-------------------------|------------------------------------------------------------------------------------------------|----------------------------------------------------------------------|
| Key Property            | The ability to uniquely identify an object through the key                                     | [Key Property](./evolvable-enums.md)                                 |
| Entity Type             |                                                                                                |                                                                      |
| Complex Type            |                                                                                                |                                                                      |
| Shared Type             | The ability to reuse a type defined by another service.                                        |                                                                      |
| Type Hierarchy          | The ability to model parent-child relationships using subtypes.                                | [Modeling with Subtypes](./Modelling%20with%20Subtypes%20Pattern.md) |
| Dictionary              | The ability to persist a variable number of properties.                                        |                                                                      |
| Evolvable Enums         | The ability to enable non-breaking changes for Enum type.                                      |                                                                      |
| Type Namespace          | The ability to reduce the need to prefix types with a qualifier to ensure uniqueness.          |                                                                      |
| Change Tracking         | The ability to get notified (push) when a change occurs in the data exposed by Microsoft Graph |                                                                      |
| Long Running Operations | The ability to model asynchronous operations.                                                  |                                                                      |
| Delta Queries           | The ability to query changes in the data exposed by Microsoft Graph                            |                                                                      |
| Navigation Properties   |                                                                                                |                                                                      |
| Viewpoint               |                                                                                                |                                                                      |
|Property projection $select||

## References

-   [Microsoft REST API
    Guidelines](https://github.com/microsoft/api-guidelines/)

-   [OData Guidelines](http://www.odata.org/documentation/)

-   [RESTful web API
    design](https://docs.microsoft.com/en-us/azure/architecture/best-practices/api-design)

-   [Microsoft Graph
    Documentation](https://developer.microsoft.com/en-us/graph/docs/concepts/overview)

-   [Microsoft Graph Explorer](https://aka.ms/ge)
