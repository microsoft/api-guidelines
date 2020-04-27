# Microsoft Azure REST API Guidelines

## History

| Date | Version | Notes |
| 2020-Mar-31 | v3.1 | 1st public release of the Azure REST API Guidelines|

## Introduction

The Azure REST API guidelines are an extension of the [Microsoft REST API guidelines][1]. Readers of this document are assumed to be also reading the [Microsoft REST API guidelines][1] and be familiar with them.  Azure guidance is a superset of the Microsoft API guidelines and services should follow them *except* where this document outlines specific differences or exceptions to those guidelines. This document does contain additional Azure-specific guidance and additional details.

### Additional guidance for Azure Resource Manager resource providers

Teams building ARM Resource Providers (RPs) MUST follow the additional guidance in the ARM Resource Provider Contract (RPC) and related documents. These documents can be found here.

* [Azure Resource Manager Wiki][2] (Internal only)
* [Azure Resource Provider Contract][3]

ARM RPs are Azure Fundamentals requirement for Azure Services and ARM RP review is another mandatory review. Some of the guidance overlaps with general API review, but passing one review will generally make the other one go very quickly.

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

1. A request is made to the default end point (GET or HEAD).  For example:

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

All Azure APIs **MUST** use explicit versioning. The Microsoft REST API guidelines offer different options on how to specify an API version and guidance on what constitutes a breaking change.  This section of the Azure API guidelines describes updates those guidelines to ensure consistency between Azure services across Azure Stack, public Azure, and sovereign clouds.

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
|========================|==============|==============|
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

Breaking changes require prior approval of the Azure REST API review board. In the case of deprecation, follow the API deprecation policy (below).  If the service is using SemVer for versioning, breaking changes constitute a major version change.

Evolutionary changes do not require prior approval (but still need a version bump).  If the service is using SemVer for versioning, evolutionary changes constitute a minor version change.

#### Handling enum additions

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

If you can foresee that an enum will be extended in the future, model the enum as an extensible enum.

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

Example request to discover versions (blob storage container list API):

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

## API deprecation policy

Disabling a runtime REST API that customers are dependent on of course has the potential of breaking their applications or services, perhaps even mission critical services.  But inevitably our APIs will become obsolete and the cost of supporting them and operating the servers on which they run will require us to deprecate and shut them down. We have a public policy that describes how we will inform customers that deprecation is coming and help them move their applications off these services and on to their replacements.

### Policy

Azure does not have a single SLA for how long we will support all services. However, we have published expectations such as [the Azure Modern Lifecycle Policy][6].  The most relevant section of the document:

> For products governed by the Modern Lifecycle Policy, Microsoft will provide a minimum of 12 months' notification prior to ending support if no successor product or service is offered—excluding free services or preview releases.

In practice, we have found this is a bare minimum of how long service endpoints must be supported. Services with any significant usage **SHOULD** expect to run until customers are no longer using them, which can be 10 years or more.

Service teams **MUST** contact the Azure API review board before communicating the deprecation externally to customers and partners (which starts the 12 month clock).

Refer to the Azure deprecation policy for more details.

#### Special case for pre-release and beta APIs

Pre-release and beta APIs are not covered by the normal API deprecation policy. Each team providing a preview API **SHOULD** communicate to customers what the policy is going to be for support and deprecation, even if that policy is “we may remove this at any time”. The Azure REST API Guidelines cover pre-release API versions. To summarize that section, they should be marked with a version tag like `2013-03-21-Preview`.

Though services may set their own deprecation policy for pre-release APIs, they should monitor these endpoints closely and consider following the normal deprecation policy and process. ** *Customers have suffered downtime because of deprecation of preview APIs* **.

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
