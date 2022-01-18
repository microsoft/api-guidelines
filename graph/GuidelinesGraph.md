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
    - [Resource Modeling Patterns](#resource-modeling-patterns)
    - [Behavior Modeling](#behavior-modeling)
    - [Error Handling](#error-handling)
  - [API contract and non-backward compatible changes](#api-contract-and-non-backward-compatible-changes)
    - [Versioning and Deprecation](#versioning-and-deprecation)
  - [Recommended API Patterns](#recommended-api-patterns)
  - [References](#references)

## 

#### History

| Date        | Notes                       |
|-------------|-----------------------------|
| 2021-Sep-28 | Using summary and patterns style. |
| 2020-Oct-04 | Initial version in Wiki.    |

## Introduction

When building a digital ecosystem API usability becomes a business priority. Success of your ecosystem depends on APIs that are easy to discover, simple to use, fit for purpose, and consistent across your products.

This document offers guidance that Microsoft Graph API producer teams MUST follow to
ensure that Microsoft Graph has a consistent and easy to use API surface. A new API design should meet the
following goals:

\- Developer friendly via consistent naming, patterns, and web standards (HTTP,
REST, JSON)

\- Work well with SDKs in many programming languages.

\- Sustainable & evolvable via clear API contracts.

The Microsoft Graph guidelines are an extension of the Microsoft REST API
guidelines. Readers are assumed also be reading and following the Microsoft REST API
guidelines except where this document outlines specific differences or exceptions to those guidelines.
Together these guidelines and a library of API patterns serve as the means by
which API teams discuss and come to consensus on API review requirements.

Technology and software are constantly changing and evolving, and as such, this
is intended to be a living document. API guidelines that change frequently lead to an uneven and inconsistent API surface. Consequently, this document will more frequently change to add guidance in areas previously uncovered, or to clarify existing guidance. It will less frequently change the directional guidance it has already provided. [Open an
issue](https://github.com/microsoft/api-guidelines/issues/new/choose) to suggest
a change or propose a new idea.

### Legend

This document offers prescriptive guidance labeled as follows:

:heavy_check_mark: **MUST** satisfy this specification. 

:no_entry: **MUST NOT** use this pattern. 

:ballot_box_with_check: **SHOULD** fulfill this specification. 

:warning: **SHOULD NOT** adopt this pattern. 

If not following these advices, you MUST disclose your reasons during the Graph API review.

## Design Approach

The design of your API is arguably the most important investment you will make. API design is what creates the first impression for developers when they discover and learn how to use your APIs. We promote API-first design approach where you begin your product design by focusing on how information will be exchanged and represented and creating an interface contract for your API which is followed by design and implementation of the backing service. This approach ensures decoupling of the interface from your implementation and is essential for agility, predictability, and reuse of your APIs. Established interface contract allows developers to use your API while internal teams are still working on implementation, API specifications enable designing of user experience and test cases in parallel. Starting with user-facing contracts also promotes a good understanding of system interactions, your modeling domain, and understanding of how the service will evolve. Microsoft Graph supports resource and query-based API styles which follow HTTP, REST, and JSON standards, where API contract is described using ODATA conventions and schema definition (see Documentation · OData - the Best Way to REST).
[Documentation · OData - the Best Way to REST](https://www.odata.org/documentation/)).

In general API design includes the following steps:

-   Define your domain model

-   Derive and name your API resources
  
-   Describe relationships between resources

-   Determine required behavior

-   Determine user roles and application permissions

-   Specify errors

When creating your API contract you will define resources based on the domain model supporting your service and identify interactions based on user scenarios. Good API design goes beyond modeling the current state of resources and it is important to plan ahead how API evolves. For this it is essential to understand and document your user scenarios as the foundation of the API design. There is no one-to-one correspondence between domain model elements and API resources because you should simplify your customer facing APIs for better usability and to obfuscate implementation details. We recommend creating a simple resource diagram, like below, to show resources and their relationships and make it easier to reason about modeling choices and the shape of your API.
![Resource model example](ModelExample.png)

After resources are defined it’s time to think about the behavior of your API which can be expressed via HTTP methods and operational resources such as functions and actions. As you think about API behavior you identify a happy path and various exceptions and deviations which will be expressed as errors and represented using HTTP codes and error messages.

At every step of your design you need to consider security, privacy and compliance as intrinsic components of your API implementation.


### Naming

API resources are typically described by nouns. Resource and property names appear in API URLs and payloads and must be descriptive and easy to understand for developers. Ease of understanding comes from familiarity and recognition  therefore when thinking about naming you should favor consistency with other Graph APIs, names in the product user interface, and industry standards. Microsoft Graph naming conventions follow [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines/).

Below is a short summary of the most often used conventions.

| Requirements                                                                                                         | Example                                                                                                                                                                                                                                                                                                                 |
|---------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| :no_entry: **MUST NOT** use redundant words in names.   |- **Right:** /places/{id}/**type** and /phones/{id}/**number** <BR> -  **Wrong** /places/{id}/*placeType* and /phones/{id}/**phoneNumber** |
| :warning: **SHOULD NOT** use brand names in type or property names.                                 | - **Right:** chat   <BR> -  **Wrong** teamsChat                                                                                              |
| :warning: **SHOULD NOT** use acronyms or abbreviations unless they are broadly understood.          | - **Right:** url or htmlSignature <BR> - **Wrong** msodsUrl or dlp                                                                        |
| :heavy_check_mark: **MUST** use singular nouns for type names.                                              | - **Right:** address  <BR> - **Wrong** addresses                                                                                           |
| :heavy_check_mark: **MUST** use plural nouns for collections (for listing a type or collection properties). | - **Right:** addresses <BR> - **Wrong** address                                                                                           |
| :ballot_box_with_check: **SHOULD** pluralize the noun even when followed by an adjective (a "postpositive").       | - **Right:** passersby or mothersInLaw    <BR> -  **Wrong** notaryPublics or motherInLaws                                                     |
| **casing** | |
| :heavy_check_mark: **MUST** use lower camel case for *all* names and namespaces                                                                                                                                                                                                      | - **Right:** automaticRepliesStatus. <BR> - **Wrong** kebab-case or snake_case.                        |
| :ballot_box_with_check: **SHOULD** case two-letter acronyms with the same case.                                                                                                                                                                                                           | - **Right:** ioLimit or totalIOAmount <BR> - **Wrong** iOLimit or totalIoAmount                        |
| :ballot_box_with_check: **SHOULD** case three+ letter acronyms the same as a normal word.                                                                                                                                                                                                 | - **Right:** fidoKey or oauthUrl <BR> - **Wrong** webHTML                                              |
| :no_entry: **MUST NOT** capitalize the word following a [prefix](https://www.thoughtco.com/common-prefixes-in-english-1692724) or words within a [compound word](http://www.learningdifferences.com/Main%20Page/Topics/Compound%20Word%20Lists/Compound_Word_%20Lists_complete.htm). | - **Right:** subcategory, geocoordinate or crosswalk <BR> - **Wrong** metaData, semiCircle or airPlane |
| :heavy_check_mark: **MUST** capitalize within hyphenated and open (spaced) compound words.                                                                                                                                                                                           | - **Right:** fiveYearOld, daughterInLaw or postOffice <BR> - **Wrong** paperclip or fullmoon           |
| **prefixes and suffixes** | |
| :heavy_check_mark: **MUST** suffix date and time properties with                                                                                  | - **Right:** dueDate — an Edm.Date <BR> - **Right:** createdDateTime — an Edm.DateTimeOffset <BR> - **Right:** recurringMeetingTime — an Edm.TimeOfDay <BR>- **Wrong** dueOn or startTime - <BR> - **Right:** instead both above are an Edm.DateTimeOffset                                                                                 |
| :ballot_box_with_check: **SHOULD** use the Duration type for durations, but if using an int, append the units.                                         | - **Right:** passwordValidityPeriod — an Edm.Duration <BR> - **Right:** passwordValidityPeriodInDays — an Edm.Int32 (NOTE use of Edm.Duration type is preferable) <BR>- **Wrong** passwordValidityPeriod — an Edm.Int32                                                                                                          |
| :no_entry: **MUST NOT** use suffix property names with primitive type names unless the type is temporal.                                          | - **Right:** isEnabled or amount <BR> - **Wrong** enabledBool                                                                                                                                                                                                                                                                |
| :ballot_box_with_check: **SHOULD** prefix property names for properties concerning a different entity.                                                 | - **Right:** siteWebUrl on driveItem, or userId on auditActor <BR> - **Wrong** webUrl on contact when its the companyWebUrl                                                                                                                                                                                                  |
| :ballot_box_with_check: **SHOULD** prefix Boolean properties with is, unless this leads to awkward or unnatural sounding names for Boolean properties. | - **Right:** isEnabled or isResourceAccount <BR>- **Wrong** enabled or allowResourceAccount <BR> - **Right:** allowNewTimeProposals or allowInvitesFrom — subjectively more natural than the examples below <BR> - **Wrong** isNewTimeProposalsAllowed or isInvitesFromAllowed — subjectively more awkward that the examples above |
| :no_entry: **MUST NOT** use 'collection', 'response', 'request ' suffixes .                                          |- **Right:** addresses <BR> - **Wrong** addressCollection                                                                                                                                                                                                          |

### Uniform Resource Locators (URLs)

A Uniform Resource Locator (URL) is how developers access the resources of your
API.

Navigation paths to Microsoft Graph resources are generally broken into multiple
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

1.  A core *user-centric concept* of the Graph, i.e. /users, /groups or /me.

2.  A Microsoft *product or service offerings* covering multiple use cases, i.e. /teamwork, /directory.

3.  A *feature offering* covering a single use case and *shared* across multiple
    Microsoft products, i.e. /search, /notifications, /subscriptions.

4.  *Administrative configuration* functions for specific products. i.e. /admin/exchange.

5.  Internal Microsoft requirements for publishing Privileged and Hidden APIs,
    routing, and load testing, i.e./loadTestEntities.

Effectively top-level categories define a perimeter for the API surface thus a
new category creation requires additional rigor and governance approval.

### Query Support

Microsoft Graph APIs should support basic query options in conformance with
OData specifications and [Microsoft REST API
Guidelines](https://github.com/microsoft/api-guidelines/blob/master/Guidelines.md#7102-error-condition-responses).
|Requirements|
|----------------------------------------------------------------------------------------------------|
| :heavy_check_mark: **MUST** support \$select on resource to enable properties projection |
| :ballot_box_with_check: **SHOULD** support \$filter with eq, ne operations on properties of entities for collections| 
| :heavy_check_mark: **MUST** support server-side pagination for collections |
| :ballot_box_with_check: **SHOULD** support pagination $top, $skip and $count for collections |

The query options part of an OData URL can be quite long, potentially exceeding
the maximum length of URLs supported by components involved in transmitting or
processing the request. One way to avoid this is to use the POST verb instead of
GET with $query segment, and pass the query options part of the URL in the request body as described
in the chapter [OData Query
Options](http://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part2-url-conventions.html#sec_PassingQueryOptionsintheRequestBody).

Limitations of \$query requests made to Microsoft Graph:

-   Microsoft Graph only supports having all the query options completely in the
    request body or completely in the request url. Graph doesn't support query
    options present in both places.

-   The parameters in \$query segment should not span multiple workloads. Support for
    \$query segment right now is limited to properties belonging to the same workload.


### Resource Modeling Patterns

You can model complex resources for your APIs using OData Entity Type or Complex Type. The main difference between these types is that Entity type declares a key property to uniquely identify its objects and  Complex Type does not. In Microsoft Graph this key property has "id" as a prescribed name.
Since objects of complex types on Graph don’t have unique identifiers, they are not directly addressable via URIs and therefore you must not use Complex Type to model addressable resources. Complex types are better suited to represent composite properties of API entities.

```XML
 <EntityType Name="Author">
    <Key>
        <PropertyRef Name="id" />
    </Key>
    <Property Name="id" Type="Edm.String" Nullable="false" />
    <Property Name="name" Type="Edm.String" />
    <Property Name="address" Type="microsoft.graph.Address" />
</EntityType>
<ComplexType Name="Address">
    <Property Name="city" Type="Edm.String" />
    <Property Name="street" Type="Edm.String" />
    <Property Name="stateOrProvince" Type="Edm.String" />
    <Property Name="country" Type="Edm.String" />
</ComplexType>
```
|  Microsoft Graph rules for modeling complex resources                                  |       |
|---------------------------------------|------------------------------------------------------------|
| :heavy_check_mark: **MUST** use String type for id      |
| :heavy_check_mark: **MUST** use a primary key composed of a single property  |
| :heavy_check_mark: **MUST** use an object as the root of all JSON payloads                               |
| :heavy_check_mark: **MUST** use a root object with  a value property to return a collection                |
| :heavy_check_mark: **MUST** include @odata.type annotations when the type is ambiguous                    |
| :warning: **SHOULD NOT** add the property id to a complex type                                              |

There are different approaches for designing an API resource model in situations
with multiple variants of a common concept. Type Hierarchy, Facets, and Flat bag
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
help to select a pattern fit for your use case.

| API qualities\   <BR> Patterns         | Properties and behavior <BR> described in metadata | Supports combinations <BR> of properties and behaviors | Simple query construction | 
|---------------------------------------------------|-------------------------------------|-----------------------------------|---------------------------|
| Type hierarchy            | yes                                        | no                                                  | no                        | 
| Facets                    | partially                                            | yes                                  | yes                       |
|Flat bag                   | no                                            | no                                  | yes                       | 




### Behavior Modeling

The HTTP operations dictate how your API behaves. The URL of an API, along with
its request/response bodies, establishes the overall contract that developers
have with your service. As an API provider, how you manage the overall request /
response pattern should be one of the first implementation decisions you make.
APIs SHOULD use resource-based designs with standard HTTP methods rather than operation resources if possible.
 Operation resources are either functions or actions. According to [ODATA standards]( http://docs.oasis-open.org/odata/odata/v4.0/errata03/os/complete/part3-csdl/odata-v4.0-errata03-os-part3-csdl-complete.html#_The_edm:Function_Element_2) a function represents an operation which returns a single instance or collection of instances of any type and doesn’t have an observable side effect. An action may have side effects and may return a result represented as a single entity or collection of any type.

|  Microsoft Graph rules for modeling behavior                                                                                          |
|-----------------------------------------------------------------------------------------------------------------|
| :heavy_check_mark: **MUST** use POST to create new entities in insertable entity sets or collections | 
| :heavy_check_mark: **MUST** use PATCH to edit updatable resources                                                 | 
| :heavy_check_mark: **MUST** use DELETE to delete deletable resources                                              | 
| :heavy_check_mark: **MUST** use GET for listing and reading resources. | 
| :warning: **SHOULD NOT** use PUT for updating resources.                                       | 
| :ballot_box_with_check: **SHOULD** avoid using multiple round trips to complete a single logical action.       | 


Bound operations must have a binding parameter matching the type of the bound resource. 
In addition both actions and functions support overloading, meaning an API definition may contain multiple actions or functions with the same name.
Microsoft Graph supports the use of optional parameters. You can use the optional parameter annotation instead of creating function or action overloads.

For a complete list of standard HTTP operations you can refer to the [Microsoft
REST API Guidelines](https://github.com/microsoft/api-guidelines/blob/master/Guidelines.md#7102-error-condition-responses).

### Error Handling

Microsoft REST API Guidelines provide guidelines that Microsoft Graph APIs should
follow when returning error condition responses. You can improve API traceability
and consistency by using recommended Graph error model and the Graph Utilities library to provide a standard implementation for your service :

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{
"error": {
    "code": "BadRequest",
    "message": "Cannot process the request because a required field is missing.",
    "target": "query",    
    "innererror": {
                "code": "RequiredFieldMissing",
                           
                }
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The top-level error code must be aligned with HTTP response status codes according to [rfc7231 (ietf.org)](https://datatracker.ietf.org/doc/html/rfc7231#section-6). 
The following examples demonstrate error modeling for common use cases:

-   **Simple error**: A workload wants to report an error with top-level details
    only. Then the error object contains the top-level error code, message and
    target (optional).

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{
  "error": {
    "code": "BadRequest",
    "message": "Cannot process the request because it is malformed or incorrect.",
	"target": "Resource X (Optional)"
  }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-   **Detailed error**: An API needs to provide service-specific details of the
    error via the innererror property of the error object. It is intended to allow
    services to supply a specific error code to help differentiate errors that
    share the same top-level error code but reported for different reasons.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{
  "error": {
    "code": "BadRequest",
    "message": "Cannot process the request because it is malformed or incorrect.",
    "innererror": {
      "code": "requiredFieldOrParameterMissing",
           
    }
  }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| Microsoft Graph enforces the following error rules                                                                    | 
|-----------------------------------------------------------------------------------------------------------------------|
| :heavy_check_mark: **MUST** return an error property with a child code property in all error responses.                 | 
| :heavy_check_mark: **MUST** return a 403 Forbidden error when insufficient scopes are present in the auth token.        | 
| :heavy_check_mark: **MUST** return a 429 Too many requests error when the caller has exceeded throttling limits.        | 
| :ballot_box_with_check: **SHOULD** return a 404 Not found error if a 403 would result in information disclosure. |

For a complete mapping of error codes to HTTP statuses you can refer to the
[rfc7231 (ietf.org)](https://datatracker.ietf.org/doc/html/rfc7231#section-6).

## API contract and non-backward compatible changes

Microsoft Graph definition of breaking changes is based on the [Microsoft REST
API
Guidelines](https://github.com/microsoft/api-guidelines/blob/graph/Guidelines.md#123-definition-of-a-breaking-change).
In general, making all but additive changes to the API contract for existing elements is
considered breaking. Adding new elements is allowed and not considered a
breaking change.

\*\* Non-breaking changes:\*\*


-   Addition of properties that are nullable or have a default value
-   Addition of a member to an evolvable enumeration 
-   Removal, rename, or change to the type of an annotation   
-   Changes to the order of properties
-   Changes to the length or format of opaque strings, such as resource IDs
-   Addition or removal of an annotation OpenType="true" 

\*\* Breaking changes:\*\*

-   Changes to the URL or fundamental request/response associated with a
    resource
-   Removal, rename, or change to an incompatible type of a declared property
-   Removal or rename of APIs or API parameters
-   Addition of a required request header
-   Addition of a EnumType members for non-evolvable enumerations
-   Addition of a Nullable="false" properties to existing types
-   Addition of a Nullable="false" parameters to existing actions and functions
-    Changes to top-level error codes
-    Introduction of server-side pagination to existing collections
-    Changes to the default order of collection elements
-    Significant changes to the performance of APIs such as increased latency, rate limits or concurrency.



### Versioning and Deprecation
As the market and technology evolves your APIs will require modifications in this case you must avoid breaking changes and add new resources and features incrementally. If that is not possible then you must version elements of your APIs.
Microsoft Graph allows versioning of elements including entities and properties. The versioning process goes along with deprecation and as soon as you introduce a new element update the previous version needs to follow the deprecation process.

You must create a new version of your element for any breaking change and name it uniquely. 
In some cases, the API will have evolved such that there is a new, natural unique name.   In other cases, the original name may still be the most descriptive for the evolved element.  In the latter case, the suffix _v2 must be added to the original name to make it unique.
The original element is then marked as deprecated using annotations.

Microsoft Graph provides two public endpoints to support API lifecycle:
1.	API sets on the v1.0 endpoint (https://graph.microsoft.com/v1.0) are in general availability (GA) status.
2.	API sets on the beta endpoint (https://graph.microsoft.com/beta) are in beta or private preview status.

Microsoft Graph APIs in the GA version guarantee API stability and consistency for its clients. If your API requires a breaking change in GA, then you MUST create new element versions and support deprecated elements for a minimum of 36 months. 
On the beta endpoint breaking changes and deprecation of APIs are allowed with consideration of dependencies and customer impact. It is best practice to test new element versions on the beta endpoint at first then promote API changes to the GA endpoint.
Detailed requirements for versioning and deprecation are described in the [Deprecation guidelines](./deprecation.md).


## Recommended API Patterns

The guidelines in previous sections are intentionally brief and provide a jump
start for Graph API developers. More detailed design guidance on REST APIs is
published at the [Microsoft REST API
Guidelines](https://github.com/microsoft/api-guidelines/) and Graph specific
patterns are outlined in the table below.

Recommended API Design patterns:

| Pattern                 | Description                                                                                    | Reference                                                            |
|-------------------------|------------------------------------------------------------------------------------------------|----------------------------------------------------------------------|
| Type Hierarchy          | The ability to model parent-child relationships using subtypes.                                | [Modeling with Subtypes](./Modelling%20with%20Subtypes%20Pattern.md) |
| Facets         | The ability to model parent-child relationships using Facet pattern.                                | [Modeling with Facets](./Modelling%20with%20Subtypes%20Pattern.md) |
                                                                     |


## References

-   [Microsoft REST API
    Guidelines](https://github.com/microsoft/api-guidelines/)

-   [OData Guidelines](http://www.odata.org/documentation/)

-   [RESTful web API
    design](https://docs.microsoft.com/en-us/azure/architecture/best-practices/api-design)

-   [Microsoft Graph
    Documentation](https://developer.microsoft.com/en-us/graph/docs/concepts/overview)

-   [Microsoft Graph Explorer](https://aka.ms/ge)
