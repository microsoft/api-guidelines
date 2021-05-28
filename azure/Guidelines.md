

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



## Introduction

The Azure REST API guidelines are an extension of the [Microsoft REST API guidelines][1]. Readers of this document are assumed to be also reading the [Microsoft REST API guidelines][1] and be familiar with them. Azure guidance is a superset of the Microsoft API guidelines and services should follow them *except* where this document outlines specific differences or exceptions to those guidelines. This document does contain additional Azure-specific guidance and additional details. While these guidelines represent and codify many years of experience building high performant, scalable cloud services on Azure, they are generally applicable to all APIs. Technology and software is constantly changing and evolving, and as such, this is intended to be a living document. [Open an issue](https://github.com/microsoft/api-guidelines/issues/new/choose) to suggest a change or propose a new idea. 


### Guideline Organization
The Guiding Principles section presents the high level concerns that affect all Azure services. Adherence to these principles creates consistent design, makes it easier for developers to use your service, and reduces the learning curve for other Azure services. All of these are critical to creating a delightful experience for Azure developers. 

These guidelines are organized in three primary sections; Advice for new services, Building Blocks, and Common API patterns. Each section builds upon the other. For example, when updating resource collections, you should make sure to understand the difference in the HTTP verbs PUT and PATCH, as their behavior is quite different and directly affects how you expose the capability of your service.  

#### Prescriptive Guidance  
This document will be as prescriptive as possible. Specific guidance is labelled and color-coded to show the relative importance.  In order from highest importance to lowest importance:

:white_check_mark: **DO** adopt this guideline or follow this pattern. If you feel you need an exception, engage with the Architecture Board prior to implementation.

:no_entry: **DO NOT** follow this pattern. If you feel you need an exception, engage with the Architecture Board prior to implementation.

:ballot_box_with_check: **YOU SHOULD** strongly consider this guideline. If not following this advice, you MUST disclose the variance during the Architecture Board design review.

:warning: **YOU SHOULD NOT** strongly consider avoiding the described pattern. If not following this advice, you MUST disclose the variance during the Architecture Board design review.

:heavy_check_mark: **YOU MAY** consider this guideline if appropriate to your situation. No notification to the architecture board is required.

*If you feel you need an exception, or need clarity based on your situation, please engage with the [API Stewardship Board] prior to release of your API.*


## Advice for new services 
Great APIs make your service usable to customers. They are intuitive, naturally reflecting and communicating the underlying model and its behavior. They lend themselves easily to client library implementations in multiple programming languages. And they don't "get in the way" of the developer, by remaining stable and predictable, especially over time.

This document provides Microsoft teams building Azure services with a set of guidelines that will help service teams build great APIs. The guidelines can be applied to create an API that is approachable, sustainable, and consistent across the Azure platform. We do this by applying a common set of patterns and web standards to the design and development of the API. For developers, a well defined and constructed API enables them to build fault tolerant applications that are easy to maintain, support, and grow. For Azure service teams, the API is often the source of code generation, and enabling a broad audience of developers across multiple languages. 

Service teams should engage the API Stewardship Board early in the development lifecycle for guidance, discussion, and review of their API. In addition, it is good practice to perform a security review, especially if you are concerned about PII leakage, compliance with GDPR, or any other considerations relative to your situation.   

Our goal is to create a developer friendly API where:
:white_check_mark: **DO** ensure that customer workloads never break
:white_check_mark: **DO** ensure that customers are able to adopt a new version of service or SDK w/out requiring code changes 

> Note: Developing a new service requires the development of at least 1 (management plane) API and potentially one or more additional (data plane) APIs.  When reviewing v1 service APIs, we see common advice provided during the review.
> A **management plane** API is implemented through the Azure Resource Manager (ARM) and is used by subscription administrators.  A **data plane** API is used by developers to implement applications.  Rarely, a subset of operations may be useful to both administrators and users, in which case it should appear in both APIs. Although the best practices and patterns described in this document apply to all REST APIs, they are especially important for **data plane** services because it is the primary interface for developers using your service. 


### Start with developer experience
A great API starts with a well thought out and designed service. It is extremely difficult, if not impossible, to create an elegant API that will work well on top of a service that is poorly designed. For example, if during a user study during a preview, you discover that customers are struggling to use your API, e.g. they don't understand the abstraction layer, take the time to fix your service. This will benefit the developer and your team. For this reason, it's important that you put yourself in the developer's shoes and think deeply about how they will be using your API and your service. 

Think about the code that a customer will write both before and after the REST API call.

:white_check_mark: **DO** provide examples in multiple languages

:white_check_mark: **DO** include at least one dynamically typed language (for example, Python or JavaScript) and one statically typed language (for example, Java or C#).


### Focus on hero scenarios
It is important to realize that writing an API is, in many cases, the easist part of providing a delightful developer experience. There are a large number of downstream activities for each API, e.g. testing, documenation, and creation of client libraries and examples.Focusing on hero scenarios reduces development, support, and maintenace costs; enables teams to align and reach consensus faster; and accelerates the time to delivery. A tell tale sign of a service that has not focused on hero scenarios is "API drift," where endpoints are inconsistent, incomplete, or juxtaposed to one another. Service teams:    

:white_check_mark: **DO** define "hero scenarios" first, then the operations required, & then design the API

:white_check_mark: **DO** provide example code that demonstrates their "Hero Scenarios."

:no_entry: **DO NOT** add APIs for speculative features customers might want 

### Start with your API definition
Understanding how your service will be used and defining its model and interaction patterns--its API--should be one of the earliest activities a service team undertakes. It should be reflect the naming decisions and make it easy for developers to implement your hero scenarios.  
:white_check_mark: **DO** provide an [OpenAPI Definition] (with [autorest extensions](https://github.com/Azure/autorest/blob/master/docs/extensions/readme.md)) that describes their service. The OpenAPI Specification is a key element of the Azure SDK plan and essential to improving the documentation, usability and discoverability of services.

:ballot_box_with_check: **YOU SHOULD** describe their services using ADL *[LINK TO ADL HERE]*. 

:ballot_box_with_check: **YOU SHOULD** use ADL to generate the required OpenAPI Definition. 

### Use previews to iterate 
 Before releasing your API, plan to invest significant design effort, get customer feedback, & iterate through multiple previews. This is especially important for V1 as it establishes the abstractions and patterns that developers will use to interact with your service. 

:ballot_box_with_check: **YOU SHOULD**  release and evaluate a minimum of 2 preview versions prior to the first GA release.  
:ballot_box_with_check: **YOU SHOULD**  create feedback loops that actively solicit feedback from preview customers.


### Avoid surprises
A major inhibitor to adoption and usage is when an API behaves in an unexpected way. Often, these are subtle design decisions that seem benign at the time, but end up introducing significant downstream friction for developers. 

:ballot_box_with_check: **YOU SHOULD** avoid polymorphism, especially in the response. An endpoint __SHOULD__ work with a single type to avoid problems during SDK creation.

:ballot_box_with_check: **YOU SHOULD** make Collections easy to work with. Collections are a common source of review comments. It is important to handle them in a consistent manner within your service. 

:ballot_box_with_check: **YOU SHOULD** return a homogeneous collection (single type).  Do not return heterogeneous collections unless there is a really good reason to do so.  If you feel heterogeneous collections are required, discuss the requirement with an API reviewer prior to implementation.

:ballot_box_with_check: **YOU SHOULD** support server-side paging, even if your resource does not currently need paging. This avoids a breaking change when your service expands.


### Design for Resiliancy 
As you build out your service and API, there are a number of decisions that can be made up front that add resiliency. Addressing these as early as possible will help you iterate faster and avoid breaking changes.

:ballot_box_with_check: **YOU SHOULD** use extensible enumerations. Extensible enumerations are modeled as strings - expanding an extensible enumeration is not a breaking change. 

:ballot_box_with_check: **YOU SHOULD** implement [conditional requests](https://tools.ietf.org/html/rfc7232) early. This allows you to support concurrency, which tends to be a concern later on.

:ballot_box_with_check: **YOU SHOULD** use wider data types (e.g. 64-bit vs. 32-bit) as they are more future-proof. For example, JavaScript can only support numbers up to 2<sup>53</sup>, so relying on the full width of a 64-bit number should be avoided.



## Building Blocks: HTTP, REST, & JSON
The Microsoft Azure Cloud platform exposes its APIs through the core building blocks of the Internet, namely HTTP, REST, and JSON. This section will provide you with a general understanding of how these technologies should be applied when creating your service. 

### HTTP
Azure services will adhere to the HTTP specification, [RFC7231](https://tools.ietf.org/html/rfc7231), as closely possible when presenting their API. This section further refines and constrains how service implementors should apply the constructs defined in the HTTP specification. It is therefore, important that you have a firm understanding of the following concepts:
* Uniform Resource Locators (URLs)
* HTTP Methods 
* Headers
* Bodies

#### URLs 
A Uniform Resource Locator (URL) is how developers will access the resources of your service. Ultimately, URLs will be how developers begin to form a cognitive model of your service. Because these will be used so heavily by developers, careful consideration should be taken when devising your structure.

:ballot_box_with_check: **YOU SHOULD** keep URLs readable and if possible, avoid UUIDs & %-encoding (ex: Cádiz)

In addition to the [URL structure guidance](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#71-url-structure) in the Microsoft REST API guidelines, Azure has specific guidance about service exposure for multi-tenant services. Specifically: 

:white_check_mark: **DO** expose their service to developers via the following URL pattern:

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

A service URL must be case-sensitive (except for scheme/host). If case doesn't match what you expect, the request __MUST__ fail with the appropriate HTTP return code. 

When returning information in a Response, services __MUST__ maintain and respect proper case values. 

Azure services __MUST NOT__ include Personal Identifying Information (PII) in the URL.

The Max length=2083 characters __MUST__ be observed. If a URL excedes this length, the service __MUST__ return a ```414-URI Too Long```

Legal characters for a URL ar: 0-9  A-Z  a-z  -  .  _  ~  /  ?  #  [  ]  @  !  $  &  '  (  )  *  +  ,  ;  =

:ballot_box_with_check: **YOU SHOULD** reserve the following characters for use exclusive use: ```/  ?  #  [  ]  @  !  $  &  '  (  )  *  +  ,  ;  =```


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
The HTTP Request / Response pattern will dictate much of how your API behaves, for example; POST methods must be idempotent, GET methods may be cached, the If-Modified and etag headers determine your optimistic concurrency strategy. The URL of a service, along with its request / response, establishes the overall contract that developers have with your service. As a service provider, how you manage the overall request / response pattern should be one of the first implementation decisions you will make. For each request / response, the service: 

:white_check_mark: **DO** validate all inputs to a request.
:white_check_mark: **DO** return the same object that was sent to the API in the response for all create or upsert operations.
 
Because beacuse information in the service URL, as well as the request / response, are strings, there must be a predictable, well-defined scheme to convert strings to their corresponding values.
:white_check_mark: **DO** use the following table when translating strings: 

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
The table below lists the request / response headers most used by Azure services Service providers.

:ballot_box_with_check: **YOU SHOULD** properly handle all headers annotated in *italics*. In addition, each request / response header: 

:white_check_mark: **DO** specify headers using kabob-style-text

:white_check_mark: **DO**  use all lowercase for headers

:no_entry: **DO NOT** use "x-" prefix for headers, unless the header already exists in production

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
Implementing services in an idempotent manner, with an "exactly once" semantic, enables developers to retry requests without the risk of unintended consequences. 

:white_check_mark: **DO** implement all operations idempotently, ideally from the outset.

:white_check_mark: **DO** adhere to the return codes in the following table when implementing your API:

Method | Description | Response Status Code 
----|----|----
GET | Read the resource | 200-OK 
DELETE | Remove the resource | 204-No Content; avoid 404-Not Found 
PATCH | Create/Modify the resource with JSON Merge Patch | 200-OK, 201-Created
PUT | Create/Replace the *whole* resource | 200-OK, 201-Created 
 
:warning: **YOU SHOULD NOT** using the POST method unless you can guarantee it can be implemented idempotently. 

#### Additional References
* [StackOverflow - Difference between http parameters and http headers](https://stackoverflow.com/questions/40492782)
* [Standard HTTP Headers](https://httpwg.org/specs/rfc7231.html#header.field.registration)
* [Why isn't HTTP PUT allowed to do partial updates in a REST API?](https://stackoverflow.com/questions/19732423/why-isnt-http-put-allowed-to-do-partial-updates-in-a-rest-api)



### REST
REST is an architectural style with broad reach that emphasizes scalability, generality, independent deployment,reduced latency via caching, and security. When applying REST to your API, you will define your service’s resources as a collections of items. These are typically the nouns you use in the vocabulary of your service. Your service's URLs determine the hierarchical path developers use to retrieve and update the state of your resource. Note, it's important to model resource state, not to behavior. There are patterns, later in these guidelines, that describe how to invoke behavior on your service. 
<span style="color:red; font-size:large">TODO: Add link to behavior section </span>

When designing your service, it is important to optimize for the developer using your API.

:white_check_mark: **DO** focus heavily on great & consistent naming

:white_check_mark: **DO** ensure your resource paths make sense

:white_check_mark: **DO** simplify call with few required query parameters & JSON fields

:white_check_mark: **DO** establish clear contracts for string values

:white_check_mark: **DO** use proper response codes/payloads so customer can self-fix


#### JSON resource schema & field mutability
For a given URL path, the JSON schema (data type) should be the same for PATCH, PUT, GET, DELETE, and GETting collection items. This allows one SDK type for input/output operations and enables the response to be passed back in request. While not explicitly defined in JSON, each field in your JSON schema should have an associated mutability rule. Tools like ADL do allow annotation of mutability, enabling more sophisticated code generation of client libraries. 

:white_check_mark: **DO** create a model of your data types. For each field, apply one of the following rules:

Field Mutability | Service Request's behavior for this bield
----| ----
**Create** | Service honors field only when creating a resource Minimize create-only fields so customers don't have to delete & re-create the resource
**Update** | Service honors field when creating or updating a resource
**Read** |Service fails request (or accept if they match what's in the resource);returns these fields in a response

#### General guidelines
The following are general guidelines when using REST.

:white_check_mark: **DO** use GET with JSON in response body

:white_check_mark: **DO** create and update resource using PATCH [RFC5789] with JSON Merge Patch request body

:white_check_mark: **DO** use PUT with JSON for wholesale create/update update operations. Take special care to hand versioning issues properly.

:white_check_mark: **DO** use DELETE when removing resources
* NOTE: Ids are "Customer Content" & Azure allows their use

:white_check_mark: **DO** make the payloads for PUT, PATCH, GET the same

:white_check_mark: **DO** make fields simple

:white_check_mark: **DO** preserve string casing/array order

:no_entry: **DO NOT** let an operation succeeed if unknown fields or bad values are passed

:no_entry: **DO NOT** return secret fields via GET
* Ex: do not return adminPassword in JSON. 
  
:heavy_check_mark: **YOU MAY**  return secret fields via POST **if absolutely necessary** 


#### Process a PATCH/PUT request
PATCH/PUT requests accept a subset of fields. Because of this, they require additional guidelines handling requests and responses. In general, you want to avoid creating partial resources as a result of create operations.

:white_check_mark: **DO** adhere to the return codes in the following table when implementing your API. These tests be processed in this oder:

When using this method |if this condition happens | use this response code
----|----|----
PATCH/PUT | Any JSON field name/value not known/valid | 422-Unprocessable Entity
PATCH/PUT | Any Read field passed (client can't set Read fields) | 422-Unprocessable Entity 
| **IF the resource does not exist** | 
PATCH/PUT | Any mandatory Create/Update field missing | 422-Unprocessable Entity 
PATCH/PUT | Create resource using Create/Update fields |201-Created 
| **If the resource already exists** |
PATCH | Any Create field doesn't match current value (allows retries) |409-Conflict 
PATCH | Update resource using Update fields | 200-OK 
PUT | Any mandatory Create/Update field missing | 422-Unprocessable Entity 
PUT | Overwrite resource entirely using Create/Update fields | 200-OK 

#### Handling Errors

:white_check_mark: **DO** deturn x-ms-error-code header with string

:white_check_mark: **DO** ensure your service returns the error response body


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


