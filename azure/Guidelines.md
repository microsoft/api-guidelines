

> ### Background
> This document is a work in progress and is intended to become an updated version of the Azure REST API guidelines. It is based on the best practices for building REST APIs, the existing Microsoft and Azure API Guidelines, and feedback from the API Stewardship Board. Your thoughts, comments, issues, pull requests, and all other forms of feedback are welcomed and encouraged. You can also reach out to Mark Weitzel as well. 
> 
> Thanks!


---
<br/>

# Microsoft Azure REST API Guidelines

## History

| Date        | Version | Notes                                               |
| ----------- | ------- | --------------------------------------------------- |
| 2020-Mar-31 | v3.1    | 1st public release of the Azure REST API Guidelines |
| 2020-Jul-31 | v3.2    | Added service advice for initial versions           |
| 2021-May-24 | WIP     | This workstream opened to update and revise the guidelines|


TODO: Add/expand section on using these guidelines for building general APIs. MS customers can use these to build their own services.



## Introduction

The Azure REST API guidelines are an extension of the [Microsoft REST API guidelines][1]. Readers of this document are assumed to be also reading the [Microsoft REST API guidelines][1] and be familiar with them. Azure guidance is a superset of the Microsoft API guidelines and services should follow them *except* where this document outlines specific differences or exceptions to those guidelines. This document does contain additional Azure-specific guidance and additional details. While these guidelines represent and codify many years of experience building high performant, scalable cloud services on Azure, they are generally applicable to all APIs. Technology and software is constantly changing and evolving, and as such, this is intended to be a living document. We welcome and encourage new ideas, input and discussion. 

<span style="color:red; font-size:large">TODO: Add sentence on how to contribute, e.g. PRs, GH discussions, etc. </span>

Developing a new service requires the development of at least 1 (management plane) API and potentially one or more additional (data plane) APIs.  When reviewing v1 service APIs, we see common advice provided during the review.
> A **management plane** API is implemented through the Azure Resource Manager (ARM) and is used by subscription administrators.  A **data plane** API is used by developers to implement applications.  Rarely, a subset of operations may be useful to both administrators and users, in which case it should appear in both APIs. Although the best practices and patterns described in this document apply to all REST APIs, they are especially important for **data plane** services because it is the primary interface for developers using your service. 




### Guideline Organization
The Guiding Principles section presents the high level concerns that affect all Azure services. Adherence to these principles creates consistent design, makes it easier for developers to use your service, and reduces the learning curve for other Azure services. All of these are critical to creating a delightful experience for Azure developers. 

These guidelines are organized in three primary sections; Guiding Principles, Building Blocks, and Common Patterns. The Guiding Principles section will present the set of considerations that impact the overall design of your API, e.g. naming, . 

Each section builds upon the other. For example, when updating resource collections, you should make sure to understand the difference in the HTTP verbs PUT and PATCH, as their behavior is quite different and directly affects how you expose the capability of your service.  

#### Prescriptive Guidance  
This document will be as prescritive as possible. Specific guideance is labelled and color-coded to show the relative importance.  In order from highest importance to lowest importance:

__MUST__ adopt this guideline or follow this pattern.

__MUST NOT__ adopt this guideline or follow this pattern.

__SHOULD__ strongly consider this guideline.

__SHOULD NOT__ strongly consider this guideline.

__MAY__ consider this guideline if appropriate to your situation.

 If you feel you need an exception, or need clarity based on your situation, please engage with the [API Stewardship Board] prior to release of your API.


## Guiding Principles 
Great APIs make your service usable to customers. They are intuitive, naturally reflecting and communicating the underlying model and its behavior. They lend themselves easily to client library implementations in multiple programming languages. And they don't "get in the way" of the developer, by remaining stable and predictable, especially over time.

This document provides Microsoft teams building Azure services with a set of guidelines that will help service teams build great APIs. The guidelines can be applied to create an API that is approachable, sustainable, and consistent across the Azure platform. We do this by applying a common set of patterns and web standards to the design and development of the API. For developers, a well defined and constructed API enables them to build fault tolerant applications that are easy to maintain, support, and grow. For Azure service teams, the API is often the source of code generation, and enabling a broad audience of developers across multiple languages. 

>Our goal is to create a developer friendly API where:
> * customer workloads __MUST__ never break
> * customers __MUST__ be able to adopt a new version of service or SDK w/out requiring code changes 

Service teams should engage the API Stewardship Board early in the development lifecycle for guidance, discussion, and review of their API. In addition, it is good practice to perform a security review, especially if you are concerned about PII leakage, compliance with GDPR, or any other considerations relative to your situation.   

### Start with developer experience
A great API starts with a well thought out and designed service. It is extremely difficult, if not impossible, to create an elegant API that will work well on top of a service that is poorly designed. For example, if during a user study during a preview, you discover that customers are struggling to use your API, e.g. they don't understand the abstraction layer, take the time to fix your service. This will benefit the developer and your team. For this reason, it's important that you put yourself in the developer's shoes and think deeply about how they will be using your API and your service. 

>Think about the code that a customer will write both before and after the REST API call. How will a developer use this API in the canonical use case?
> * Service teams __SHOULD__ provide examples in multiple languages, and __SHOULD__ include at least one dynamically typed language (for example, Python or JavaScript) and one statically typed language (for example, Java or C#).


### Focus on hero scenarios
We all want to get our service out into the wild as quickly as possible. We don't want to waste time and energy building things our customers never use, or that rare edge case that happens once in a blue moon. While it sounds obvious, approach the design from the customer to the service; not from service to the customer. 

It is important to realize that writing an API is, in many cases, the easist part of providing a delightful developer experience. There are a large number of downstream activities for each API, e.g. testing, documenation, and creation of client libraries and examples.Focusing on hero scenarios reduces development, support, and maintenace costs; enables teams to align and reach consensus faster; and accelerates the time to delivery. A tell tale sign of a service that has not focused on hero scenarios is "API drift," where endpoints are inconsistent, incomplete, or juxtaposed to one another. Service teams:    
> * __SHOULD__ define "hero scenarios" first, then the operations required, & then design the API
> * __SHOULD__ provide example code that demonstrates their "Hero Scenarios."
> * __SHOULD NOT__ add APIs for speculative features customers might want

### Reflect key concepts through naming
<span style="color:red; font-size:large">TODO: Intro sentence for context </span>

As you identify the tasks and activities that developers will accomplish using your service, it will be important to develop a vocabulary that intuitively reflects your core concepts. 
 * Start with the "things" your API manipulates, then think about the operations that a developer needs to do to these "things".
  * Keep the verbs present-tense.  Avoid the use of past or future tense in most cases.
  * Avoid the use of generic names like "Object", "Job", "Task", "Operation" (for example - the list is not exhaustive).
  * What happens to the names when the focus of the service expands?  It may be worth starting with a less generic name to avoid a breaking change later on.
  * 
<span style="color:red; font-size:large">TODO: Pull in section from Heath's doc </span>


### Start with your API definition
You don't build a house without a blueprint. Neither should you build your service without a well thought-out API definition. Understanding how your service will be used and defining its model and interaction patterns--its API--should be one of the earliest activities a service team undertakes. It should be reflect the naming decisions and make it easy for developers to implement your hero scenarios.  
> * All Services **MUST** provide an [OpenAPI Definition] (with [autorest extensions](https://github.com/Azure/autorest/blob/master/docs/extensions/readme.md)) that describes their service. The OpenAPI Specification is a key element of the Azure SDK plan and essential to improving the documentation, usability and discoverability of services.
> * All Services __SHOULD__ describe their services using ADL *[LINK TO ADL HERE]*. 
> * ADL __SHOULD__ be used to generate the required OpenAPI Definition. 


### Use previews to iterate 
 Before releasing your API, plan to invest significant design effort, get customer feedback, & iterate through multiple previews. This is especially important for V1 as it establishes the abstractions and patterns that developers will use to interact with your service.

>Use previews to get the shape right. There are different notification, breaking change, and lifetime requirements on preview API versions.
> * Service teams __SHOULD__ release and evaluate a minimum of 2 preview versions prior to the first GA release.  
> * Service teams __SHOULD__ create feedback loops that actively solicit feedback from preview customers.

<span style="color:red; font-size:large">TODO: Provide references on how to run an effective preview </span>

### Avoid surprises
A major inhibitor to adoption and usage is when an API behaves in an unexpected way. Often, these are subtle design decisions that seem benign at the time, but end up introducing significant downstream friction for developers. 
* Avoid polymorphism. An endpoint should work with a single type to avoid problems during SDK creation. Remember that a change to the model is a breaking change.
* Make Collections easy to work with. Collections are a common source of review comments. It is important to handle them in a consistent manner within your service. 
  * A collection __SHOULD__ return a homogeneous collection (single type).  Do not return heterogeneous collections unless there is a really good reason to do so.  If you feel heterogeneous collections are required, discuss the requirement with an API reviewer prior to implementation.
  * A collection __SHOULD__ support server-side paging, even if your resource does not currently need paging. This avoids a breaking change when your service expands.

* Build in [idempotency](https://stripe.com/blog/idempotency) from the outset. In a distributed cloud, each HTTP call must be idempotent. In fact, the HTTP specification requires that GET, PUT, DELETE, and HEAD be idempotent. You must be resilient in the face of failure. Developers rely on idempotency to build fault-tolerant systems.
  * Example: Allow the developer to use PUT or PATCH to create a resource with a user-specified name or ID.  A developer will commonly want to download a specific resource by name or ID.  This provides the developer with an idempotent mechanism for creating resources.


### Design for Resiliancy 
As you build out your service and API, there are a number of decisions that can be made up front that add resiliancy. Addressing these as early as possible will help you iterate faster and avoid breaking changes.
* Use extensible enumerations. Extensible enumerations are modeled as strings - expanding an extensible enumeration is not a breaking change. 
* Implement conditional requests early. This allows you to support concurrency, which tends to be a concern later on.
* If your API specifies access conditions to another resource:
** Think about how to represent that model polymorphically. For example, you may be using a SQL Azure connection now, but extend to Cosmos DB, Azure Data Lake, or Redis Cache later on. Think about how you can specify that resource in a non-breaking manner.
* Implement managed identity access controls for accessing the other resource. Do not accept connection strings as a method of specifying access permissions.

<span style="color:red; font-size:large">TODO: I'd like to be much more prescriptive here. </span>
* Be concerned about data widths of numeric types.  Wider data types (e.g. 64-bit vs. 32-bit) are more future-proof.
* Think about how the interface will be represented by an SDK.  For example, JavaScript can only support numbers up to 2<sup>53</sup>, so relying on the full width of a 64-bit number should be avoided.

### Avoid breaking changes
Innovation and design improvements to a service and its API are often the result of user studies and understanding how a service is used through telemetry. Well defined and manage previews enable rapid learning through a tight feedback loop. Being able to iterate quickly and incorporate these learnings can be a competitve differentor. However, as services mature, developers will rely more and more on the API, using them as a foundation for their own applications. For this reason, it is imperative to effectively manage change. This is especially true once a service has GA'd its API. The guidelines in this document have been designed in such a way as to help service teams balance innovation and stability.

> Service teams __MUST__ follow the breaking change guidelines specified in this document. *[LINK TO SECTION]*


There are additional considerations for management APIs. Microsoft teams building this aspect of the service should refer to the following resources for supplemental guidelines:
* [Azure Resource Manager Wiki][2] (Microsoft only).
* Use [RPaaS](https://armwiki.azurewebsites.net/rpaas/overview.html) (Microsoft only) to implement the Azure Resource Provider.


## Building Blocks: HTTP, REST, & JSON
The Microsoft Azure Cloud platform exposes its APIs through the core building blocks of the Internet, namely HTTP, REST, and JSON. This section will provide you with a general understanding of how these technologies should be applied when creating your service. 

### HTTP
Azure services will adhere to the HTTP specification, [RFC7231](https://tools.ietf.org/html/rfc7231), as closely possible when presenting their API. This section further refines and constrains how service implementors should apply the constructs defined in the HTTP specification. It is therefore, important that you have a firm understanding of the following concepts:
* Uniform Resource Locators (URLs)
* HTTP Methods 
* Headers
* Bodies

#### URLs 
A Uniform Resource Locator (URL) is how developers will access the resources of your service. Ultimately, URLs will be how developers begin to form a cognitive model of your service. Becuase these will be used so heavily by developers, careful consideration should be taken when devising your structure. For these reasons, service providers __SHOULD__ keep URLs readable and if possible, avoid UUIDs & %-encoding (ex: Cádiz)

In addition to the [URL structure guidance](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#71-url-structure) in the Microsoft REST API guidelines, Azure has specific guidance about service exposure for multi-tenant services. Specifically: 

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

##### Additional URL considerations
When services produce URLs in response headers or bodies, they **MUST** use a consistent form – either always a GUID for tenant identifier or always a single verified domain - regardless of the URL used to reach the resource.

It is common that resources will differ by case. In addition, case may also affect computed values. Therefore, a service URL must be case-sensitive (except for scheme/host). If case doesn't match what you expect, the request __MUST__ fail with the appropriate HTTP return code. Further, when you are returning information in a Response, services __MUST__ maintain and respect proper case values. 

Logging of URLs is another common practice. We want to take every precaution to prevent the leakage of Personal Identifying Information (PII). Azure services __MUST NOT__ include Personal Identifying Information (PII) in the URL.

The Max length=2083 characters __MUST__ be observed. If a URL excedes this length, the service __MUST__ return a ```414-URI Too Long```

Legal characters for a URL ar: 0-9  A-Z  a-z  -  .  _  ~  /  ?  #  [  ]  @  !  $  &  '  (  )  *  +  ,  ;  =
Services __SHOULD__ reserve the following characters for use exclusive use: ```/  ?  #  [  ]  @  !  $  &  '  (  )  *  +  ,  ;  =```


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


### HTTP Request / Response Pattern
The HTTP Request / Response pattern will dictate much of how your API behaves, for example; POST methods must be idempotent, GET methods may be cached, the If-Modified and etag headers determine your optimistic concurrency strategy. The URL of a service, along with its request / response, establishes the overall contract that developers have with your service. As a service provider, how you manage the overall request / response pattern __SHOULD__ be one of the first implementation decisions you will make. For each request / response, the service: 

> * __MUST__ validate all inputs to a request.
> * __SHOULD__ return the same object that was sent to the API in the response for all create or upsert operations.
 
Because beacuse information in the service URL, as well as the request / response, are strings, there must be a predictable, well-defined scheme to convert strings to their corresponding values. Service provides __MUST__ use the following table when translating strings: 

Data type | Document string must be
-------- | -------
Boolean  | true / false 
Integer  | -253+1 to +253-1 (limit due to IEEE-754 [RFC8259]())https://datatracker.ietf.org/doc/html/rfc8259) 
Float    | [IEEE-754 binary64](https://en.wikipedia.org/wiki/Double-precision_floating-point_format) 
String   | (Un)quoted?, max length, case-sensitive, multiple delimiter 
UUID     | {}? casing? hyphens? [RFC4122](https://datatracker.ietf.org/doc/html/rfc4122)
Date/Time (Header) | [RFC1123](https://datatracker.ietf.org/doc/html/rfc1123)
Date/Time (Query parameter) | YYYY-MM-DDTHH:mm:ss.sssZ [RFC3339](https://datatracker.ietf.org/doc/html/rfc3339) 
Byte array | Base-64 encoded, max length 

<span style="color:red; font-size:large">TODO: Expand the explanation for numbers. </span>

#### Common Request & Response Headers
The table below lists the request / response headers most used by Azure services Service providers __SHOULD__ properly handle all headers annotated in *italics*. In addition, each request / response header: 
> * __MUST__ be specificed using kabob-style-text
> * __MUST__ be all lowercase
> * __SHOULD NOT__ use "x-" prefix, unless already existing in production


Header Key |	Applies to |	Example 
------------ | ------------- | -------------
*authorization*	 | Request |	Bearer eyJ0...Xd6j (Support Azure Active Directory) 
*x-ms-useragent*  |  Request | [see Telemetry](http://TODO:link-goes-here)
traceparent | Request | [see Distributed Tracing]](http://TODO:link-goes-here)
tracecontext | Request | [see Distributed Tracing](http://TODO:link-goes-here)
accept | Request | application/json
if-match | Request | "67ab43" or * (no quotes) (see Conditional Access)
if-none-match | Request | "67ab43" or * (no quotes) [see Conditional Access](http://TODO:link-goes-here)
If-Modified-Since | | Request | (RFC1123) [see Optimistic Concurrency](http://TODO:link-goes-here)
If-Unmodified-Since | Request | (RFC1123) [see Optimistic Concurrency](http://TODO:link-goes-here)
date [RFC1123] | Both | Sun, 06 Nov 1994 08:49:37 GMT 
*content-type* | Both | application/merge-patch+json
*content-length* | Both | 1024
*x-ms-request-id* | Response | [see Customer Support](http://TODO:link-goes-here)
etag | Response | "67ab43" [see Conditional Access](http://TODO:link-goes-here)
retry-after | Response | 180 [see Throttling Client Requests]
*x-ms-error-code* | Response | [see Processing a REST Request](http://TODO:link-goes-here)
Last-Modified | Response | (RFC1123) [see Optimistic Concurrency](http://TODO:link-goes-here)


<span style="color:red; font-size:large">TODO: Fix the links. </span>


#### HTTP methods & idempotency
[Idempotentency](https://www.lexico.com/en/definition/idempotent), or the ability for the state of resource to remain unchanged when the same operation is applied, is a fundamental property of resilient cloud services. Implementing services in an idempotent manner, with an "exactly once" semantic, enables developers to retry requests without the risk of unintended consequences. (See our guiding principle of "No surprises".) 


Method | Description | Response Status Code 
----|----|----
GET | Read the resource | 200-OK 
DELETE | Remove the resource | 204-No Content; avoid 404-Not Found 
PATCH | Create/Modify the resource with JSON Merge Patch | 200-OK, 201-Created
PUT | Create/Replace the *whole* resource | 200-OK, 201-Created 

> * Service providers __MUST__ implement all operations idempotently. 
> * Service providers __SHOULD__ avoid using POST unless it can be implemented idempotently. 

#### Additional References
* [StackOverflow - Difference between http parameters and http headers](https://stackoverflow.com/questions/40492782)
* [Standard HTTP Headers](https://httpwg.org/specs/rfc7231.html#header.field.registration)
* [Why isn't HTTP PUT allowed to do partial updates in a REST API?](https://stackoverflow.com/questions/19732423/why-isnt-http-put-allowed-to-do-partial-updates-in-a-rest-api)



### REST




#### Handling unknown properties or parameters
This should dovetail nicely into the added guidance on how to deal with "readOnly" values that a client may (incorrectly) supply in a request.



### JSON

## Common API Patterns

### Performing an Action

### Collections

### Long Running Operations

### API Versioning

### Distributed Tracing & Service Telemetry

### Jobs
* e.g. cascading delete

### Bring your own storage
* Getting data into your service
* Working with blobs

### Optimistic concurrency


## Final Thoughts / Summary
* Careful consideration up front
* Long term decisions that are often codified in SDKs, CODE, etc.
* Reach out and engage the stewardship team!


## API Guidelines Quick Reference Sheet
<span style="color:red; font-size:large">TODO: Should we create a quick reference sheet??</span>
Add the Must / Must NOT w/links
See if we can generate this