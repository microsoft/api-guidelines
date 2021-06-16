# Microsoft Azure REST API Guidelines

## History

| Date        | Version | Notes                                               |
| ----------- | ------- | --------------------------------------------------- |
| 2020-Mar-31 | v3.1    | 1st public release of the Azure REST API Guidelines |
| 2020-Jul-31 | v3.2    | Added service advice for initial versions           |

## Introduction

The Azure REST API guidelines are an extension of the [Microsoft REST API guidelines][1]. Readers of this document are assumed to be also reading the [Microsoft REST API guidelines][1] and be familiar with them.  Azure guidance is a superset of the Microsoft API guidelines and services should follow them *except* where this document outlines specific differences or exceptions to those guidelines. This document does contain additional Azure-specific guidance and additional details.

### Additional guidance for Azure Resource Manager resource providers

Teams building ARM Resource Providers (RPs) MUST follow the additional guidance in the ARM Resource Provider Contract (RPC) and related documents. These documents can be found here.

* [Azure Resource Manager Wiki][2] (Internal only)
* [Azure Resource Provider Contract][3]

ARM RPs are Azure Fundamentals requirement for Azure Services and ARM RP review is another mandatory review. Some of the guidance overlaps with general API review, but passing one review will generally make the other one go very quickly.

### Advice for new services

Developing a new service requires the development of at least 1 (management plane) API and potentially one or more additional (data plane) APIs.  When reviewing v1 service APIs, we see common advice provided during the review.

> A **management plane** API is implemented through the Azure Resource Manager (ARM) and is used by subscription administrators.  A **data plane** API is used by developers to implement applications.  Rarely, a subset of operations may be useful to both administrators and users, in which case it should appear in both APIs.

* Think about naming from the context of a **developer experience**.  
  * Start with the "things" your API manipulates, then think about the operations that a developer needs to do to these "things".
  * Keep the verbs present-tense.  Avoid the use of past or future tense in most cases.
  * Avoid the use of generic names like "Object", "Job", "Task", "Operation" (for example - the list is not exhaustive).
  * What happens to the names when the focus of the service expands?  It may be worth starting with a less generic name to avoid a breaking change later on.
* Think about the code that a customer will write both before and after the REST API call.  How will a developer use this API in the canonical use case?
  * Consider multiple languages, and include at least one dynamically typed language (for example, Python or JavaScript) and one statically typed language (for example, Java or C#).
* Use previews to get the shape right.  There are different notification, breaking change, and lifetime requirements on preview API versions.
  * We recommend a minimum of 2 preview versions prior to your first GA release.  However, there is no hard rule for previews.  
  * You should gather feedback from your customers and iterate until the API is useful.  Actively solicit feedback from your preview customers.

Preventing future breaking changes is a source of concern during initial review.  Without a history, the review process attempts to identify patterns that may result in breaking changes later on.  For instance, think about the following:

* [Think about idempotency](https://stripe.com/blog/idempotency).  In a distributed cloud, each HTTP call must be idempotent.  You must be resilient in the face of failure.  A developer must rely on idempotency to build fault-tolerant systems.
  * The HTTP specification requires that GET, PUT, DELETE, and HEAD be idempotent.
  * Prefer allowing the developer to use PUT or PATCH to create a resource with a user-specified name or ID.  A developer will commonly want to download a specific resource by name or ID.  This provides the developer with an idempotent mechanism for creating resources.
* Collections are a common source of review comments:
  * Return collections with server-side paging, even if your resource does not currently need paging.  This avoids a breaking change when your service expands.
  * A collection should return a homogeneous collection (single type).  Do not return heterogeneous collections unless there is a really good reason to do so.  If you feel heterogeneous collections are required, discuss the requirement with an API reviewer prior to implementation.
  * Think about how the developer can reason about the collection organization.  Filtering is a common customer request.
* Avoid polymorphism.  An endpoint should work with a single type to avoid problems during SDK creation.  Remember that a change to the model is a breaking change.
* Use extensible enumerations unless you are completely sure that the enumeration will never expand.  Extensible enumerations are modeled as strings - expanding an extensible enumeration is not a breaking change.
* Implement [conditional requests](https://tools.ietf.org/html/rfc7232) early.  This allows you to support concurrency, which tends to be a concern later on.
* If your API specifies access conditions to another resource:
  * Think about how to represent that model polymorphically.  For example, you may be using a SQL Azure connection now, but extend to Cosmos DB, Azure Data Lake, or Redis Cache later on.  Think about how you can specify that resource in a non-breaking manner.
  * Implement managed identity access controls for accessing the other resource.  Do not accept connection strings as a method of specifying access permissions.
* Be concerned about data widths of numeric types.  Wider data types (e.g. 64-bit vs. 32-bit) are more future-proof.
* Think about how the interface will be represented by an SDK.  For example, JavaScript can only support numbers up to 2<sup>53</sup>, so relying on the full width of a 64-bit number should be avoided.
* Implement (and encourage) the use of PATCH for resource modifications.  
  * The PATCH operation should be able to modify any mutable property on the resource.
  * Prefer JSON merge-patch ([RFC 7396](https://tools.ietf.org/html/rfc7396)) over JSON patch ([RFC 6902](https://tools.ietf.org/html/rfc6902)) as the accepted data format for PATCH operations.
* Get a security review of your API.  You should be especially concerned with PII leakage, GDPR compliance and any other compliance regulations appropriate to your situation.

Additionally, for management APIs:

* Follow the advice in the [Azure Resource Manager Wiki][2] (internal only).
* Use [RPaaS](https://armwiki.azurewebsites.net/rpaas/overview.html) (internal only) to implement the Azure Resource Provider.

## API definition

All Services **MUST** provide an [OpenAPI Definition] (with [autorest extensions](https://github.com/Azure/autorest/blob/master/docs/extensions/readme.md)) that describes their service. The OpenAPI Specification is a key element of the Azure SDK plan and essential to improving the documentation, usability and discoverability of services.

## URL structure

In addition to the [URL structure guidance](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#71-url-structure) in the Microsoft REST API guidelines, Azure has specific guidance about service exposure for multi-tenant services

All services **MUST** expose their service to developers via the following URL pattern:

```text
https://<service>.<cloud-instance>/<unit-of-multi-tenancy>/<service-defined-root>
```

Where:

* **service** - the name of the service such as "blobstore", "servicebus", "directory", or "management"
* **cloud-instance** - the DNS domain name at the root of the cloud instance.  For instance, public Azure uses `azure.net`.  Sovereign clouds uses different domains.
* **service-defined-root** - the root of the service-specific path, such as "blobcontainer", "myqueue", etc.
* **unit-of-multi-tenancy** - refers to a globally unique moniker that identifies a unique container in the Azure service that has the following properties:

  * This container is the boundary of isolation between different tenants of the service.
  * Quotas as set and enforced at the level of this container - but there will be different limits for different operations; and operations will be service specific.
  * Resources in the service are attached to this container and are tied to this container in terms of lifecycle. For example someone signs up, they get this container. If they unsubscribe (or don’t pay their bills) then cleanup of this container occurs and the resources associated with this container are cleaned up. Cleanup follows a state machine – the container and the resources attached to it are deactivated first (and can be easily restored if required), and if no response for some period then deleted.
  * It is the container for billing – which means the owner of this container sees one bill for the resource usage of all azure services under this container’s identifier.

For Azure PaaS services like SQL Azure, Azure Storage, Caching, etc., the unit of multi-tenancy is the Azure subscription id, which is a GUID. This ensures consistent access using the same URL pattern, and identifier, across all these services.

When services produce URLs in response headers or bodies, they **MUST** use a consistent form – either always a GUID for tenant identifier or always a single verified domain - regardless of the URL used to reach the resource.

### Direct endpoint URLs

In addition to the required format above, services **MAY** also choose to expose direct endpoint for performance or routing reasons.  The direct endpoint should be discoverable by clients, to ensure that developers are presented with a consistent pattern for accessing Azure services.

The format of the root of the direct endpoint **MUST** be as follows:

```text
https://<tenant-id>-<service-defined-root>.<service>.azure.net
```

1. A request is made to the default endpoint (GET or HEAD).  For example:

   ```text
   GET https://blobstore.azure.net/contoso.com/account1/container1/blob2
   ```

2. That request is returned with the `Content-Location` header set to the direct endpoint.  See [RFC2557]:

   ```text
   200 OK
   Content-Location: https://contoso-dot-com-account1.blobstore.azure.net/container1/blob2
   ```

   Or, with the GUID format:

   ```text
   200 OK
   Content-Location: https://00000000-0000-0000-C000-000000000046-account1.blobstore.azure.net/container1/blob2
   ```

## Versioning

All Azure APIs **MUST** use explicit versioning. The Microsoft REST API guidelines offer different options on how to specify an API version and guidance on what constitutes a breaking change.  This section of the Azure API guidelines describes updates to those guidelines to ensure consistency between Azure services across Azure Stack, public Azure, and sovereign clouds.

Retirement of an API version must follow the standard [_Azure Global Retirements and Breaking Changes_][7] policies in effect.

### Specifying the version in Azure

The Microsoft REST API guidelines give two options for how services and clients communicate the version: a url segment and a query parameter. Azure services **MUST** use the api-version query parameter. For example:

```text
GET https://blobstore.azure.com/foo.com/acct1/c1/blob2?api-version=1.0
PUT https://blobstore.azure.com/foo.com/acct1/c1/b2?api-version=2014-12-07
POST https://blobstore.azure.com/foo.com/acct1/c1/b2?api-version=2015-12-07
```

### API Changes that require a version change

There are three groups of changes that may happen to an API.

1. Changes made to an **EXISTING** API version due to security or compliance reasons.  We shall refer to these types of changes as _Compliance changes_.
2. Changes made to an API that may cause a client making the API call to fail, such as removal of an endpoint or property or changing the format of the body.  We refer to these types of changes as _Breaking changes_.
3. Additive changes made to an API that do not cause a client making the API call to fail, such as the addition of a new optional property or a new endpoint.  We refer to these types of changes as _Evolutionary changes_.

With the exception of _Compliance changes_ (which are extremely rare), Azure services **MUST** update the version number of their API whenever there is a change to the API, no matter how small.  Customers will "lock the API version" so that their code does not fail when the service introduces new features.  They rely on the fact that an API version is a contract with the services that will never change.

A _breaking change_ is any change in the API that may cause client or service code making the API call to fail. Obvious examples of such a change are the removal of an endpoint, adding or removing a required field or changing the format of the body (from XML to JSON for example). Even though we recommend clients ignore new fields, there are many libraries and clients that fail when new fields are introduced. Removing an endpoint from an API is always a _breaking change_.  Adding a new endpoint is always an _evolutionary change_.  Changes to properties may be _evolutionary_ or _breaking_ depending on the type of change and whether the change is to an input parameter or output parameter:

| Property change        | Input        | Output       |
|:-----------------------|:------------:|:------------:|
| Remove a property      | Breaking     | Breaking     |
| Add optional property  | Evolutionary | Breaking     |
| Add required property  | Breaking     | Breaking     |
| Data type change       | Breaking     | Breaking     |
| Format change          | Breaking     | Breaking     |
| Integer widens         | Evolutionary | Breaking     |
| Integer narrows        | Breaking     | Evolutionary |
| Add new value to enum  | Evolutionary | Breaking     |
| Remove value from enum | Breaking     | Breaking     |
| Optional to required   | Breaking     | Breaking     |
| Required to optional   | Evolutionary | Breaking     |

Breaking changes require prior approval of the Azure REST API review board and approval through the [Azure Global Breaking Change Policy][7]. In the case of deprecation, follow the [Azure Global Retirement Policy][7].  If the service is using SemVer for versioning, breaking changes constitute a major version change.

Evolutionary changes do not require prior approval (but still need a version bump).  If the service is using SemVer for versioning, evolutionary changes constitute a minor version change.

#### Changing the API without changing the version

Because the API version represents a contract that a developer can rely on when generating SDKs to communicate with the service, there are a limited set of situations where changing the API is permissible without a version bump.  The only changes universally allowed:

1. Adding a new (optional) value to an extensible enum.

An extensible enum is (in essence) a string.  The values of the extensible enum drive intellisense and documentation, but the values are not considered exhaustive.

If a service is **ONLY** available in the Azure public cloud, then an additional situation can be used to add functionality without changing the version:

1. Adding a new (optional) query parameter to adjust the output (for example, adding filter options to a list operation).
2. Adding optional computed read-only output values that are generated based on the new (optional) query parameters.

For example, let's say an image service wants to add bounding-box information to the output of an operation.  The service can add a new query parameter `includeBoundingBox=true` and then include the bounding box information within the output only when the new query parameter is specified.  A version bump is recommended, but not required.  If not changing the API version, the service **MUST** update all data centers before the new query parameter is advertised to customers.

> **DO NOT** use this mechanism just to get around the version bump.  Adding such query parameters results in sub-optimal API designs and should only be used for exceptional circumstances.

This functionality is only available for single cloud deployments because the API version specifies the contract with the developer.  Consider, for example, if such a functionality was included in Azure public cloud and not a sovereign cloud.  A developer creating an SDK based on this functionality may see the application work in one cloud but fail when targeting the other despite using the same API version in both cases.  For the purposes of this situation, "other clouds" includes Azure Stack and other deployment mechanisms such as containers for on-premise usage.

Do not add computed output values if the computed value can be calculated from other information in the payload.  It unnecessarily expands the payload.

All situations where the API definition is changed (irrespective of whether a version change happens or not) **MUST** be reviewed by the Azure REST API Review Board before release.  

### Preview Versions

Preview versions of the API can be indicated by adding the suffix `-preview.X` to the end of the API version, where `X` is an incrementing integer.  For example:

* `2020-05-01-preview.1`
* `1.0-preview.2`

Preview versions are not treated the same way as release versions.  In general, there are two types of previews:

* **Private** previews are released to a known subset of users.  The service team knows how to contact each person within the private preview.  There are no restrictions on changes within a private preview, as long as the service team communicates effectively with their users on what changes are made and when they will be made.

* **Public** previews are released broadly, but contain APIs that may change between previews and may be deleted prior to the final version.  The `X` integer (indicating the revision of the preview) must be incremented for all breaking changes (resulting in a new API version).  Evolutionary changes may be added as needed, as long as the change is communicated broadly.

You should follow the axiom "don't surprise your customers" when deciding whether to increment the preview version, and err on the side of incrementing the preview version.

Preview APIs [follow a different path for deprecation and retirement](https://dev.azure.com/msazure/AzureWiki/_wiki/wikis/AzureWiki.wiki/37683/Retirement-of-Previews), and are not subject to the same guarantees as APIs marked as generally available.  Preview APIs have a life-span of not more than 12 months, after which they must be retired. 

#### Why Azure recommends conservative API versioning

Azure history is replete with anecdotes that directly relate to API versioning.  For instance, Cognitive Services unintentionally broke customers by making changes to the API structure without a version bump with updates that they did not think would be breaking changes.  These changes led customers to question the stability and maturity of the product and increased the churn rate for the services.

Even changes that are evolutionary can cause problems.  For instance, let's say that a service adds a new feature via a new endpoint in the API.  The SDK gets updates to support this new API, but the service does not bump the version number.  Since the roll out of the new feature is not atomic, there is a period of time (potentially months long) where the feature is available in some regions but not others.  A customer has the potential for attempting to use the feature in two different regions and having it work in one region but not the other, despite the two regions supporting the same version number.  This is only made worse when we consider Azure Stack, which can be upwards of a year behind the public cloud offerings.

There are a few mechanisms that can reduce breaking changes and their effects on our customers.

#### Use PATCH instead of PUT for updates

The HTTP PUT verb is an idemopotent apply operation for the API version being used.  The [Microsoft API Guidelines already recommend the use of PATCH](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#742-patch) for updates.

Consider the following sequence:

* User1 creates a resource with version v2, using a new optional parameter.
* Later, User2 wants to update the resource using unrelated settings.  Using version v1, User2 issues a GET, does the changes, and then issues a PUT to replace the resource definition.

In this case, the optional parameter is lost because of the replace semantics.  The optional parameter only exists on API version v2, and not on version v1.

Service teams SHOULD prefer and recommend PATCH operations for updating resources.

#### Use extensible enums

While removing a value from an enum is a breaking change, adding an enum can be handled with an _extensible enum_.  An extensible enum is a string value that has been marked with a special marker - setting `modelAsString` to true within an `x-ms-enum` block.  For example:

```json
"createdByType": {
   "type": "string",
   "description": "The type of identity that created the resource.",
   "enum": [
      "User",
      "Application",
      "ManagedIdentity",
      "Key"
   ],
   "x-ms-enum": {
      "name": "createdByType",
      "modelAsString": true
   }
}
```

Always model an enum as a string unleess you are positive that the symbol set will **NEVER** change over time.

### Group versioning in Azure and Azure Stack

Azure Stack allows customers and hosters to deploy their own small versions of Azure and upgrade it at a different pace than Azure. In order to make it possible to write an application or SDK that targets both Azure and Azure Stack, additional versioning policy is necessary. Contact the Azure Stack team for further guidance.

### Version discovery

Simpler clients may be hardcoded to a single version of a service. Since Azure services offer each version for a well-known period of time, a client that’s regularly maintained can be always operational without further complexity as long as during regular maintenance the client is moved forward to new versions in advance of older ones being retired.

API version discovery is needed when either a given hosted service may expose a different API version to different clients (e.g. latest API version only available in certain regions or to certain tenants) or the service itself may exist in different instances (e.g. a service that may be run on Azure or hosted on-premises). In both of those cases clients may get ahead of services in the API version they use. In might also be possible for a client version to ship ahead of its corresponding service update, leading to the same situation. Lastly, version discovery is useful for clients that want to warn operators that an API they depend on may expire soon.

Azure services **SHOULD** support API version discovery.  If they support it:

1. Services **MUST** support HTTP `OPTIONS` requests against all resources, including the root URL for a given tenant or the global root if no tenant identity is tracked or not a multi-tenant service
2. Services **MUST** include the `api-supported-versions` header, containing a comma-separated list of versions conforming to the Azure versioning scheme. This list must include all group versions as well as all major-minor versions supported by the target resource. For cases where no specific version applies (e.g. sometimes the root resource), the list still must contain the group versions supported by the service.
3. If a given service supports versions of the API that are known to be planned for deprecation in a year or less, it **MUST** include those versions (group and major.minor) in the `api-deprecated-versions` header.
4. In addition to the functionality described here, services **MAY** support HTTP `OPTIONS` requests for other purposes such as further discovery, CORS, etc.
5. Services **MAY** allow unauthenticated HTTP `OPTIONS` requests. When doing so, authors need to consider whether HTTP `OPTIONS` requests against non-existing resources result in 404s and whether that is leaking sensitive information. Certain scenarios, such as support for CORS pre-flight requests, require allowing unauthenticated HTTP `OPTIONS` requests.
6. For services that do rolling updates where there is a point in time where some front-ends are ahead of others version-wise, all front-ends **MUST** report the previous version as the latest version until the rolling update covers all instances and only then switch over to reporting the new latest version. This ensures that clients will not detect a version and then get load-balanced into a front-end that does not support it yet.
7. If using OData and addressing an expanded resource, the HTTP `OPTIONS` request **SHOULD** return the group versions that are supported across the expanded set.

Example request to discover API versions (blob storage container list API):

```text
OPTIONS /?comp=list HTTP/1.1
host: accountname.blob.core.azure.net
```

Example response:

```text
200 OK
api-supported-versions: 2011-08,2012-02,1.1,2.0
api-deprecated-versions: 2009-04,1.0
Content-Length: 0
```

Clients that use version discovery are expected to cache version information. Since there’s a year of lead time after an API version shows in the `api-deprecated-versions` before it’s removed, checking once a week should provide sufficient lead time to client authors or operators. In the rare case where a server rolls back a version that clients are already using, the service will reject requests because they are ahead of the latest version supported. Whenever a client sees a `version-too-new` error, it should re-execute its version discovery procedure.

## Long running operations

The Microsoft REST API guidelines for Long Running Operations are an updated, clarified and simplified version of the Asynchronous Operations guidelines from the 2.1 version of the Azure API guidelines. Unfortunately, to generalize to the whole of Microsoft and not just Azure, the HEADER used in the operation was renamed from `Azure-AsyncOperation` to `Operation-Location`. Services **SHOULD** support both `Azure-AsyncOperation` and `Operation-Location` HEADERS, even though they are redundant so that existing SDKs and clients will continue to operate. Clients that call these services **SHOULD** look for both HEADERS and prefer the `Operation-Location` version. Both HEADERS **MUST** return the same value.

## Retiring pre-release and beta APIs

Pre-release and beta APIs are not covered by the Azure Global Retirement and Deprecation Policy. Each team providing a preview API **SHOULD** communicate to customers what the policy is going to be for support and deprecation, even if that policy is “we may remove this at any time”. The Azure REST API Guidelines cover pre-release API versions. To summarize that section, they should be marked with a version tag like `2013-03-21-Preview`.

Though services may set their own deprecation policy for pre-release APIs, they should monitor these endpoints closely and consider following the normal deprecation policy. **Customers have suffered downtime because of deprecation of preview APIs**.

<!-- Links -->
[1]: https://github.com/microsoft/api-guidelines
[RFC2557]: https://www.ietf.org/rfc/rfc2557.txt

<!-- Azure ARM Links -->
[2]: https://aka.ms/armwiki
[3]: https://github.com/Azure/azure-resource-manager-rpc

<!-- Open API Spec -->
[OpenAPI Specification]: https://github.com/Azure/adx-documentation-pr/wiki/Getting-started-with-OpenAPI-specifications

<!-- Versioning Guidelines -->
[6]: https://support.microsoft.com/en-us/help/30881
[7]: http://aka.ms/aprwiki
