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

#### Asynchronous operations

The OneAPI guidelines for Long Running Operations guidelines are an updated, clarified and simplified version of the Asynchronous Operations guidelines from the 2.1 version of the Azure API guidelines. Unfortunately, to generalize to the whole of Microsoft and not just Azure, the board decided to rename the HEADER used in the operation from `Azure-AsyncOperation` to `Operation-Location`. Services can and **SHOULD** support both `Azure-AsyncOperation` and `Operation-Location` HEADERS, even though they are redundant so that existing SDKs and clients will continue to operate. Clients that call these services **SHOULD** look for both HEADERS and prefer the `Operation-Location` version. Both HEADERS **MUST** return the same value.

### Additional guidance for Azure Resource Manager resource providers

Teams building ARM RPs MUST follow the additional guidance in the ARM RPC and related documents. These documents can be found here.

* [Azure Resource Manager Wiki][2] (Internal only)
* [Azure Resource Provider Contract][3]

ARM RPs are a CEC requirement for Azure Services and ARM RP review is another mandatory review. Some of the guidance overlaps with general API review, but passing one review will generally make the other one go very quickly.

## Swagger to describe API

All Services **MUST** provide Swagger that describes their service. Swagger is a key element of the Azure SDK plan and essential to improving the usability and discoverability of services. Swagger is a CEC requirement for Azure Services.

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

When services produce URLs in response headers or bodies, they **MUST** use a consistent form – either always a GUID for tenant identifier or always a single verified domain - regardless of the URL used to reach the resource.

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

At a high level, changes to the contract of an API constitute a breaking change. Changes that impact backwards compatibility of an API is also considered a breaking change. Teams MAY define backwards compatibility as their business needs require. For example, Azure defines the addition of a new JSON field in a response to be not backwards compatible. Anything that would violate the Principle of Least Astonishment is considered a breaking change in Azure. Below are some concrete examples of what constitutes a breaking change. In the below breaking change scenarios, the API version must be changed.

#### Existing property is removed

If a property called `foo` that was present in v1 of the API needs to be removed, it must be done in a newer API version.

#### New property added to response

If a new property/field is added to the response an API, the GET-PUT pipeline will be broken. Consider the case where from portal a customer updates the value of a new property "A". Another customer does a GET of this resource using the SDK. The SDK will ignore the property since it does not understand it. From the SDK, the customer does a PUT using the model that was returned from the GET. This will overwrite the change made by the first customer from the portal.

#### New required property added to request

If a new property is made required in the request body, clients will have no way to set this and the request will fail.

#### Property name has changed

Note that this is implied by the requirement that adding/and removing properties are breaking changes, but in some ways worse, since it leads to the possibility of reusing a property name. Even with an API version change, this change is discouraged because it creates documentation and cognitive challenges.

#### Property type has changed

Property `foo was a boolean in v1 but is changed to a string. A client using the existing API version tries to set it as a boolean, but the service will fail since its now expecting a string. So, the API version must be updated.

#### Property default value has changed

If a property is optional and the service provides a default value, changing that default requires an updated API version.

#### Allowed values for an enum have changed
Enum “foo” had allowed values as “val1” and “val2” in v1 of API. Now, the values accepted by the service are “val1”, “val2” and “val3”.  The client will fail to de-serialize if “val3” comes back in the response.

#### API has been removed or renamed

V1 of API contract supported `PUT /resourceType1/{resourceType1_name}` but the service no longer supports this method. This scenario should follow the proper Azure API deprecation policy and must be done in an updated API version.

#### Behavior of existing API has changed

There is a functional change in what the API was doing. This is a complex issue because it sometimes is not an easy option to maintain the old behavior even on an old API version. It also is very confusing to end users even when version is update and documented. Behavior changes need to be well justified and discussed on a case-by-case basis.

#### Error contracts have changed

#### Property is made required (from optional)

If property “foo” was optional in the request body of v1 and now it is required, this should result in an API version change. If not changed, clients relying on the older API version will fail if this property is not passed.

#### URL format has changed

Resource parameter names change from `/resourceType1/{resourceType1_name}` to `/resourceType1/{resourceType1_id}`. This will impact code generation.

#### Resource naming rules should not change

This could result in failures which would have earlier succeeded. Even if the rules become less strict, clients relying on earlier name constraints to perform local validation will fail.

### Non-Breaking Changes

The following changes are considered backwards compatible and hence non-breaking.

#### Adding new APIs to an existing service

When a new resource types is added, it does not require API version to be updated for existing types.

#### Bug fixes to existing API

Bug fixes to existing API which don’t fall into one of the above categories of breaking changes as described above are fine.

### Group versioning in Azure and Azure Stack

Azure Stack allows customers and hosters to deploy their own small versions of Azure and upgrade it at a different pace than Azure. In order to make it possible to write an application or SDK that targets both Azure and Azure Stack, additional versioning policy is necessary. Contact the Azure Stack team for further guidance.

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

Azure does not have a single SLA for how long we will support all services. However, we have published expectations such as [the Azure Cloud Lifecycle FAQ][6].  The most relevant section of the document:

> Azure Cloud Services will support no fewer than the latest two SDK versions for deploying new Cloud Services. Microsoft will provide notification 12 months before retiring a SDK in order to smooth the transition to a supported version.

In practice, we have found this is a bare minimum of how long service endpoints must be supported. Services with any significant usage **SHOULD** expect to run until customers are no longer using them, which can be 10 years or more.

Service teams **MUST** contact the Azure API review board before communicating the deprecation externally to customers and partners (which starts the 12 month clock).

Refer to the Azure deprecation policy for more details.

#### Special case for pre-release and beta APIs

Pre-release and beta APIs are not covered by the normal API deprecation policy. Each team providing a preview API **SHOULD** communicate to customers what the policy is going to be for support and deprecation, even if that policy is “we may remove this at any time”. The Azure REST API Guidelines cover pre-release API versions. To summarize that section, they should be marked with a version tag like `2013-03-21-Preview`.

Though services may set their own deprecation policy for pre-release APIs, they should monitor these endpoints closely and consider following the normal deprecation policy and process. ** *Customers have suffered downtime because of deprecation of preview APIs* **. In some cases the code was written by consultants or employees without any awareness on behalf of the customer.

<!-- Links -->
[1]: https://github.com/microsoft/api-guidelines
[RFC2557]: http://www.ietf.org/rfc/rfc2557.txt

<!-- Azure ARM Links -->
[2]: https://aka.ms/armwiki
[3]: https://github.com/Azure/azure-resource-manager-rpc

<!-- Versioning Guidelines -->
[6]: http://support.microsoft.com/gp/azure-cloud-lifecycle-faq
