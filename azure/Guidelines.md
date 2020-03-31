# Microsoft Azure REST Design Guidelines

## History

<dl>
<dd>2020-Mar-31 v3.0</dd>
<dt>Version 3.0 of the Azure REST API Guidelines was copied from Sharepoint
into Markdown and uploaded to GitHub as the basis for future improvements.</dt>
</dl>

## Introduction

The Azure REST API guidelines are an extension of the Microsoft-wide [OneAPI guidelines][1] (which historically drew heavily from an earlier version of the Azure REST API guidelines). Readers of this document are assumed to be also reading the [OneAPI guidelines][1] and be familiar with them.  Azure guidance is a superset of OneAPI guidelines and services should follow them *except* where this document outlines specific differences or exceptions to those guidelines. This document does contain additional Azure-specific guidance and additional details.

The OneAPI guidelines are available on Github at [https://github.com/microsoft/api-guidelines][1].

Highlights of the OneAPI guidelines include:

* **REST fundamentals**.  Guidance around REST basics such as HTTP headers, verbs, status codes, and other similar areas.
* **Versioning**.  The guidance for versioning covers the externally facing mechanics of versioning and compatibility guarantees between versions.  Guidance for when API owners should increment versions is also present.
* **CORS**.  Guidance around the use of CORS.
* **Authentication**.  Guidance around the authentication and its use.
* **Encoding**.  Guidance around the use of JSON encoding and standards.
* **Error responses**.  A standard format for error responses is closed.
* **Dates and times**.  Detailed guidance around date and time formats and encoding is provided.
* **Relationship to OData**.  Guidance around the relationship between OData and REST APIs.
* **JSON representation**.  JSON is preferred over XML and other formats.
* **Long running operations**.  The OneAPI guidelines are updated and simplified over the previous Azure guidelines.
* **Push notifications via Webhooks**.

### Updating from previous versions of the Azure API guidelines

The OneAPI guidelines started from the last revision of the Azure API Guidelines and are highly compatible. Azure services that are already using an earlier version of the guidelines should not require much additional work.

#### Asynchronous operations

The OneAPI guidelines for Long Running Operations guidelines are an updated, clarified and simplified version of the Asynchronous Operations guidelines from the 2.1 version of the Azure API guidelines. Unfortunately, to generalize to the whole of Microsoft and not just Azure, the board decided to rename the HEADER used in the operation from `Azure-AsyncOperation` to `Operation-Location`. Services can and **SHOULD** support both `Azure-AsyncOperation` and `Operation-Location` HEADERS, even though they are redundant so that existing SDKs and clients will continue to operate. Clients that call these services **SHOULD** look for both HEADERS and prefer the `Operation-Location` version. Both HEADERS **MUST** return the same value.

### Additional guidance for Azure Resource Manager resource providers

Teams building ARM RPs MUST follow the additional guidance in the ARM RPC and related documents. These documents can be found here.

* [Azure Resource Manager Documents][2] (Internal only)
* [Azure Resource Provider Contract][3] (Internal only)

ARM RPs are a CEC requirement for Azure Services and ARM RP review is another mandatory review. Some of the guidance overlaps with general API review, but passing one review will generally make the other one go very quickly.

## Swagger to describe API

All Services **MUST** provide Swagger that describes their service. Swagger is a key element of the Azure SDK plan and essential to improving the usability and discoverability of services. Swagger is a CEC requirement for Azure Services

## URL structure

In addition to the URL structure guidance in the OneAPI guidelines, Azure has specific guidance about service exposure for multi-tenant services

### URL structure

All services **MUST** expose their service to developers via the following URL pattern:

```
http(s)://<service>.azure.net/<unit-of-multi-tenancy>/<service-defined-root>
```

Where:

* **service** - the name of the service such as "blobstore", "servicebus", "directory", or "management"
* **service-defined-root** - the root of the service-specific path, such as "blobcontainer", "myqueue", etc.
* **unit-of-multi-tenancy** - refers to a globally unique moniker that identifies a unique container in the Azure service that has the following properties:

  * This container is the boundary of isolation between different tenants of the service.
  * Quotas as set and enforced at the level of this container - but there will be different limits for different operations; and operations will be service specific.
  * Resources in the service are attached to this container and are tied to this container in terms of lifecycle. For example someone signs up, they get this container. If they unsubscribe (or don’t pay their bills) then cleanup of this container occurs and the resources associated with this container are cleaned up. Cleanup follows a state machine – the container and the resources attached to it are deactivated first (and can be easily restored if required), and if no response for some period then deleted.
  * It is the container for billing – which means the owner of this container sees one bill for the resource usage of all azure services under this container’s identifier.

For Azure PaaS services like SQL Azure, Azure Storage, Caching, etc., the unit of multi-tenancy is the Azure subscription id, which is a GUID. This ensures consistent access using the same URL pattern, and identifier, across all these services.

When services produce URLs in response headers or bodies, they **MUST** use a consistent form – either always a GUID for tenant identifier or always a single verified domain - regardless of the URL used to reach the resource

### Direct endpoint URLs

In addition to the required format above, services **MAY** also choose to expose direct endpoint for performance or routing reasons.  The direct endpoint should be discoverable by clients, to ensure that developers are presented with a consistent pattern for accessing Azure services.

The format of the root of the direct endpoint **MUST** be as follows:

```
http(s)://<tenant-id>-<service-defined-root>.<service>.azure.net
```

1. A request is made to the default end point (GET or HEAD).  For example:

   ```
   GET https://blobstore.azure.net/contoso.com/account1/container1/blob2
   ```

2. That request is returned with the `Content-Location` header set to the direct endpoint.  See [RFC2557]:

   ```
   200 OK
   Content-Location: http://contoso-dot-com-account1.blobstore.azure.net/container1/blob2
   ```

   Or, with the GUID format:

   ```
   200 OK
   Content-Location: http://00000000-0000-0000-C000-000000000046-account1.blobstore.azure.net/container1/blob2
   ```

## Versioning

All Azure APIs **MUST** support explicit versioning.  It's critical that clients can count on services to be stable over time, and it's critical that Azure services can add features and make changes.

The OneAPI guidelines offer a couple of different options on how to specify an API version and guidance on what constitutes a breaking change.  This section of the Azure API guidelines describes which options are required of Azure services as well as some guidance about deprecation policy.  There is also a section about additional versioning practices necessary to support Azure Stack and Azure compatibility.

Further discussion on versioning principles and criteria can be found in the [Azure API versioning guidelines][4] (Internal only).

### Specifying the version in Azure

The OneAPI guidelines give two options for how services and clients communicate the version: a url segment and a query parameter. Azure services **MUST** use the api-version query parameter. For example:

```
GET http://blobstore.azure.com/foo.com/acct1/c1/blob2?api-version=1.0
PUT http://blobstore.azure.com/foo.com/acct1/c1/b2?api-version=2014-12-07
POST http://blobstore.azure.com/foo.com/acct1/c1/b2?api-version=2015-12-07
```

### Breaking changes in Azure

A breaking change is any change in the API that may cause client or service code making the API call to fail. Obvious examples of such a change are the removal of an endpoint, adding or removing a required field or changing the format of the body (from XML to JSON for example).

Even though we recommend clients ignore new fields, there are many libraries and clients that are strict. Therefore, Azure services **MUST** update the version number of their API even when adding optional fields. In fact, servers should be as strict as possible. Ignoring a field can result in the API accepting content that containered a typo or an element at the wrong level of nesting. If this missing field changes the semantics (for example, we have seen cases where security settings were misplaced and ignored, leaving the resources more exposed than intended) this can be a huge and hard to discover error.

### Group versioning in Azure and Azure Stack

Azure Stack allows customers and hosters to deploy their own small versions of Azure and upgrade it at a different pace than Azure. In order to make it possible to write an application or SDK that targets both Azure and Azure Stack, additional versioning policy is necessary. This guidance is still being worked on, but the latest can be found here: [Azure and Azure Stack Versioning][5].

### Version discovery

Simpler clients may be hardcoded to a single version of a service. Since Azure services offer each version for a well-known period of time, a client that’s regularly maintained can be always operational without further complexity as long as during regular maintenance the client is moved forward to new versions in advance of older ones being retired.

API version discovery is needed when either a given hosted service may expose a different API version to different clients (e.g. latest API version only available in certain regions or to certain tenants) or the service itself may exist in different instances (e.g. a service that may be run on Azure or hosted on-premises). In both of those cases clients may get ahead of services in the API version they use. In might also be possible for a client version to ship ahead of its corresponding service update, leading to the same situation. Lastly, version discovery is useful for clients that want to warn operators that an API they depend on may expire soon.

Azure services **SHOULD** support API version discovery.  If they support it:

1. Services **MUST** support HTTP `OPTIONS` requests against all resources, including the root URL for a given tenant or the global root if no tenant identity is tracked or not a multi-tenant service
2.	Services **MUST** include the `api-supported-versions` header, containing a comma-separated list of versions conforming to the Azure versioning scheme. This list must include all group versions as well as all major-minor versions supported by the target resource. For cases where no specific version applies (e.g. sometimes the root resource), the list still must contain the group versions supported by the service.
3.	If a given service supports versions of the API that are known to be planned for deprecation in a year or less, it must include those versions (group and major.minor) in the `api-deprecated-versions` header.
4.	In addition to the functionality described here, services may support HTTP `OPTIONS` requests for other purposes such as further discovery, CORS, etc.
5.	Services may allow unauthenticated HTTP `OPTIONS` requests. When doing so authors need to consider whether HTTP `OPTIONS` requests against non-existing resources result in 404s and whether that is leaking sensitive information. Certain scenarios, such as support for CORS pre-flight requests, require allowing unauthenticated HTTP `OPTIONS` requests.
6.	For services that do rolling updates where there is a point in time where some front-ends are ahead of others version-wise, all front-ends should report the previous version as the latest version until the rolling update covers all instances and only then switch over to reporting the new latest version. This ensures that clients will not detect a version and then get load-balanced into a front-end that does not support it yet.
7.	If using OData and addressing an expanded resource, the HTTP `OPTIONS` request should return the group versions that are supported across the expanded set.

Example request to discover versions (blob storage container list API):

```
OPTIONS /?comp=list HTTP/1.1
host: accountname.blob.core.azure.net
```

Example response:

```
200 OK
api-supported-versions: 2011-08,2012-02,1.1,2.0
api-deprecated-versions: 2009-04,1.0
Content-Length: 0
```

Clients that use version discovery are expected to cache version information. Since there’s a year of lead time after an API version shows in the `api-deprecated-versions` before it’s removed, checking once a week should provide sufficient lead time to client authors or operators. In the rare case where a server rolls back a version that clients are already using, the service will reject requests because they are ahead of the latest version supported. Whenever a client sees a `version-too-new` error, it should re-execute its version discovery procedure.

## API deprecation policy

Disabling a runtime REST API that customers are dependent on of course has the potential of breaking their applications or services, perhaps even mission critical services.  But inevitably our APIs will become obsolete and the cost of supporting them and operating the servers on which they run will require us to deprecate and shut them down. We have a public policy that describes how we will inform customers that deprecation is coming and help them move their applications off these services and on to their replacements.

### Policy

Azure does not have a single SLA for how long we will support all services. However, we have published expectations such as [here][6].  The most relevant section of the document: 

> Azure Cloud Services will support no fewer than the latest two SDK versions for deploying new Cloud Services. Microsoft will provide notification 12 months before retiring a SDK in order to smooth the transition to a supported version.

In practice, we have found this is a bare minimum of how long service endpoints must be supported. Services with any significant usage **SHOULD** expect to run until customers are no longer using them, which can be 10 years or more.

#### Special case for pre-release and beta APIs

Pre-release and beta APIs are not covered by the normal API deprecation policy. Each team providing a preview API **SHOULD** communicate to customers what the policy is going to be for support and deprecation, even if that policy is “we may remove this at any time”. The Azure REST API Guidelines cover pre-release API versions. To summarize that section, they should be marked with a version tag like `2013-03-21-Preview`.

Though services may set their own deprecation policy for pre-release APIs, they should monitor these endpoints closely and consider following the normal deprecation policy and process. ** *Customers have suffered downtime because of deprecation of preview APIs* **. In some cases the code was written by consultants or employees without any awareness on behalf of the customer.

## Process

This section describes the process of implementing this policy.  This process should:

* Minimize surprise. Customers, internal partners, management, the SDK and documentation team.  All should be surprised as little as possible by the deprecation of an API.
* Be lightweight and simple to implement

The key to minimizing surprise is to communicate the decision to deprecate an API as early and as widely as possible. Because the policy guarantees a minimum of 12 months between notification and actual retirement, there should be plenty of time to communicate and adjust plans based on feedback or changing conditions.

Basically the process should be an iteration of “make a decision to proceed”; “communicate the decision”, “evaluate the feedback” and “iterate” on whether to move to the next step of the process or to change the decision or the timeframe.  This is a flexible process, but the suggested steps are:

### Step 1: Make a business decision to deprecate the API

The decision to deprecate a REST API (or tool or client library etc.) is a business decision and should be made by the responsible business unit based on factors such as usage data, the degree to which the service is mission critical, potential for data loss or customer downtown, and the costs of continuing to support and operate the service. In the spirit of the rest of this process, the decision to deprecate a REST API should come as little surprise:  it should be obvious by that point that there is a newer version of the service that is widely available and offers advantages over the version being deprecated, and plenty of time should have passed for customers to update to the new version (at least by the end of the 12 month retirement grace window described by policy). There is no requirement of executive approval for API deprecation, but in the spirit of reducing surprise, communication with management is encouraged.

### Step 2: Document the decision

As decision is being made, it should be documented.  The documentation should answer such questions as:

* What versions of which APIs are being deprecated?
* What versions of which APIs replace those being deprecated?
* Who is impacted by deprecating these APIs?  Which internal partners?  What customers?
* Why are the APIs being deprecated?  Support cost?  COGS?  Security or other reasons?
* When exactly will the APIs be deprecated?

### Step 3: Communicate with major partners and customers

Many services will have particularly important customers or partners dependent on the versions of the APIs being deprecated. The team should make an effort to contact these early in the process and get their feedback. Ideally the feedback from these contacts can still be used to reevaluate the decision to deprecate an API or evaluate the appropriateness of the replacement APIs.

### Step 4: Communicate to the Azure REST API Oversight and Azure SDK stakeholders

The Azure CEC REST API review board, Azure SDK and documentation teams should be informed via e-mail of the decision. There should be a short window of time (2 work days) for these teams to provide feedback. Silence should be interpreted as assent to move to the next step. These teams may suggest coordinating multiple deprecating APIs from multiple teams in order to streamline the release and communication of changes.

### Step 5: Communicate widely internally

The decision should be communicated widely within Azure.  Mail describing the decision should be sent to the Azure API review board, WW – Communities and other appropriate aliases. There should be another short (2-5 working days) feedback period before moving on to the next step.

### Step 6: Communicate externally, starting the 12 month clock

The decision next **MUST** be communicated to customers and partners. The Service Notices site gives instructions for communicating this and other service changes. The Playbook contains a link to a submission form to request e-mail be sent.

https://microsoft.sharepoint.com/teams/azurecomms/SitePages/Email-Request.aspx

Detail:

> **Sunsetting/retirement**: These emails are sent when a service or feature is going to be deprecated/retired. For services or features that are generally available, we must provide at least 1 years’ notice, and then we send regular reminders to the impacted customers to remind them that the service is being retired (typical cadence is: 1 year prior to end date, reminders: 6 months prior, 3 months prior, 6 weeks, and then weekly, depending on number of customers who are still impacted). For services or features in preview, stakeholders should consider the customer impact, how long it will take to migrate customers to another service or feature, and if there is feature parity with the new service or feature. In addition, stakeholders should get LCA review/approval of their sunsetting/retirement communication strategy.

It is important to remember that these notifications need to be localized.

### Step 7: Update the SDKs and documentation

#### REST API documentation

The supported REST API group version(s) will be indicated on the [REST API reference landing page for the particular service][8] with a table similar to below:

| REST API | Notes |
| **XXX Service** | |
| 2011-01-23+ | Supported |
| 2010-11-01  | Deprecated - April 23, 2013 |

Alternatively, customers can go to the reference for a [specific API][9] and see whether the particular API is in the 12-month deprecation window.  Once the API has been removed, the reference documentation will be removed, as well.

#### Client libraries

NuGet packages become deprecated no later than when the corresponding REST APIs become deprecated.  This will be indicated in the following ways:

1. **On the NuGet gallery**: A deprecation note and date will be added to the description for the NuGet package.
2. **On azure.com**: A table (grouped by programming language) similar to below will be updated:

| Client Library | REST Version | Note |
| **XXX Client** ||
| 1.7 | 2011-01-23 ||
| 1.6 | 2010-11-01 | Deprecated - April 23, 2013 |

This package will be also linked to from other relevant documentation pages on azure.com, MSDN, etc.

#### Tools and "All in one" SDK bundles

Tools become deprecated no later than when *any* of the corresponding REST APIs the tools depend on also become deprecated.  This will be indicated on azure.com with a table similar to below:

| Tools | Note |
| **Azure SDK for .NET (with VS Tooling)** ||
| 2.0+ | Supported |
| October 2012 | Deprecated - April 23, 2013 |
|||
| **Azure PowerShell ** ||
| 0.6.12+ | Supported |
| 0.6.11 | Deprecated - April 23, 2013 |

This page will be linked to from other relevant documentation pages.

### Step 8: Start reporting deprecated version of the API to discovery APIs

Section 9.5 of the REST API guidelines describes a mechanism for discovering supported versions of APIs. It includes a mechanism for reporting deprectated APIs:

*	If a given service supports versions of the API that are known to be planned for deprecation in a year or less, it **MUST** include those versions (group and major.minor) in the `api-deprecated-versions` header.

At this point in the process, services should start using this mechanism.

### Step 9: Continue to monitor the REST API endpoints during the retirement period

During the 12 month retirement window, the team should continue to monitor the use of the REST API and take feedback from customers. If conditions change or new feedback brings the original decision or timing into question, the team should be prepared to cancel or postpone deprecation. If the decision is changed, the same Steps 1 through 7 above should be followed to communicate the updated decision.

### Step 10: Turn off the REST endpoint

After the 12 month retirement grace period has passed, the team can turn off their REST API during the normal process of updating their services.

<!-- Links -->
[1]: https://github.com/microsoft/api-guidelines
[2]: http://sharepoint/sites/AzureUX/Sparta/Shared%20Documents/Forms/AllItems.aspx?RootFolder=%2fsites%2fAzureUX%2fSparta%2fShared%20Documents%2fSpecs&FolderCTID=0x012000BE9C61C2BE4C444E8018AB8C82DE4CF9
[3]: http://sharepoint/sites/AzureUX/Sparta/Shared%20Documents/Specs/Resource%20Provider%20API%20v2.docx?d=wcc855351793a4417a536ab992218ad55
[4]: https://microsoft.sharepoint.com/teams/azure-arc/SitePages/API%20Versioning.aspx
[5]: https://microsoft.sharepoint.com/teams/AzureStack/_layouts/15/WopiFrame.aspx?sourcedoc=%7bD5665517-0CDF-4E9C-9B9A-D053140E3FA7%7d&file=Azure%20and%20Azure%20Stack%20API%20Versioning.docx
[6]: http://support.microsoft.com/gp/azure-cloud-lifecycle-faq
<!-- BEGIN: no longer being actively maintained -->
[7]: https://microsoft.sharepoint.com/teams/azurecomms/SitePages/Email-Request.aspx
[8]: https://docs.microsoft.com/en-us/previous-versions/azure/ee460799(v=azure.100)
[9]: https://docs.microsoft.com/en-us/previous-versions/azure/reference/ee460813(v=azure.100)?redirectedfrom=MSDN
<!-- END: no longer being actively maintained -->

[RFC2557]: http://www.ietf.org/rfc/rfc2557.txt

