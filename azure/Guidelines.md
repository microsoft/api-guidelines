

> ### Background
> This document is a work in progress and is intended to become an updated version of the Azure REST API guidelines. It is based on the best practices for building REST APIs, the existing Azure API Guidelines, and feedback from the API Stewardship Board. Your thoughts, comments, pull requests, and all other forms of feedback are welcomed and encouraged. If you have any questions, please reach out to @Mark Weitzel. 
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

<span style="color:red">TODO: Add sentence on how to contribute, e.g. PRs, GH discussions, etc. </span>

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
<span style="color:red">TODO: Intro sentence for context </span>

As you identify the tasks and activities that developers will accomplish using your service, it will be important to develop a vocabulary that intuitively reflects your core concepts. 
 * Start with the "things" your API manipulates, then think about the operations that a developer needs to do to these "things".
  * Keep the verbs present-tense.  Avoid the use of past or future tense in most cases.
  * Avoid the use of generic names like "Object", "Job", "Task", "Operation" (for example - the list is not exhaustive).
  * What happens to the names when the focus of the service expands?  It may be worth starting with a less generic name to avoid a breaking change later on.
  * 
<span style="color:red">TODO: Pull in section from Heath's cognitive doc </span>


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

<span style="color:red">TODO: Provide references on how to run an effective preview </span>

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

<span style="color:red">TODO: I'd like to be much more prescriptive here. </span>
* Be concerned about data widths of numeric types.  Wider data types (e.g. 64-bit vs. 32-bit) are more future-proof.
* Think about how the interface will be represented by an SDK.  For example, JavaScript can only support numbers up to 2<sup>53</sup>, so relying on the full width of a 64-bit number should be avoided.

### Avoid breaking changes
Innovation and design improvements to a service and its API are often the result of user studies and understanding how a service is used through telemetry. Well defined and manage previews enable rapid learning through a tight feedback loop. Being able to iterate quickly and incorporate these learnings can be a competitve differentor. However, as services mature, developers will rely more and more on the API, using them as a foundation for their own applications. For this reason, it is imperative to effectively manage change. This is especially true once a service has GA'd its API. The guidelines in this document have been designed in such a way as to help service teams balance innovation and stability.

> Service teams __MUST__ follow the breaking change guidelines specified in this document. *[LINK TO SECTION]*



## Building Blocks: HTTP, REST, & JSON
Purpose: Understand the building blocks. 
Communicate the core concepts. 
The more common patterns that APIs use are all built using these. 





There are additional considerations for management APIs. Microsoft teams building this aspect of the service should refer to the following resources for supplemental guidelines:
* [Azure Resource Manager Wiki][2] (Microsoft only).
* Use [RPaaS](https://armwiki.azurewebsites.net/rpaas/overview.html) (Microsoft only) to implement the Azure Resource Provider.

### HTTP

#### Request / Response

REQUEST
``` 
POST /items?color=orange HTTP/1.1
host: www.contoso.com:443
accept: application/json
content-type: application/json 
{ "someValue": true} 
```

RESPONSE
```
HTTP/1.1 200 OK
etag: "511GaciaHb28"
content-length: 23
content-type: application/json
{"someValue": true}
```

> A service __MUST__ validate all inputs
> A service __MUST NOT__ include PII in the URL

##### Common Request & Response Headers

Header Key |	Applies to |	Example 
------------ | ------------- | -------------
authorization	 | Request |	Bearer eyJ0...Xd6j (Support Azure Active Directory) 
x-ms-useragent  |  Request | (see Telemetry)
traceparent | Request | (see Distributed Tracing)
tracecontext | Request | (see Distributed Tracing)
accept | Request | application/json
if-match | Request | "67ab43" or * (no quotes) (see Conditional Access)
if-none-match | Request | "67ab43" or * (no quotes) (see Conditional Access)
date [RFC1123] | Both | Sun, 06 Nov 1994 08:49:37 GMT 
content-type | Both | application/merge-patch+json
content-length | Both | 1024
x-ms-request-id | Response | (see Customer Support)
etag | Response | "67ab43" (see Conditional Access)
retry-after | Response | 180 (see Throttling Client Requests)
x-ms-error-code | Response | (see Processing a REST Request)


#### URLs 

#### Idempotency

 **Building a cloud service**
Customers can create fault-tolerant apps by supporting retries/idempotency
Remaining fault-tolerant in the face of failures
Network requests fail for many reasons
Unhandled exception, hardware failure, scale-down, code upgrade, orchestrator VM balancing, timeout, server throttling, network outage
Bottom line: a client may not get a service's response
Client code must retry to compensate for these failures. So, services must implement operations idempotently

Exactly Once --> Client Retries & Service Idempotency

> The problem with POST

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


## Final Thoughts / Summary
* Careful consideration up front
* Long term decisions that are often codified in SDKs, CODE, etc.
* Reach out and engage the stewardship team!