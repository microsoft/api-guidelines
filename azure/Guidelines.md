
> ### Background
> This document is a work in progress and is intended to become an updated version of the Azure REST API guidelines. It is based on the best practices for building REST APIs, the existing Microsoft and Azure API Guidelines, and feedback from the API Stewardship Board. Your thoughts, comments, issues, pull requests, and all other forms of feedback are welcomed and encouraged. You can also reach out to Mark Weitzel as well. 
> 
> Thanks!


---
<br/>

# Microsoft Azure HTTP/REST API Guidelines

## Introduction

These guidelines offer prescriptive guidance that Azure service teams MUST follow ensuring that customers have a great experience by designing APIs meeting these goals:
- Developer friendly via consistent patterns & web standards (HTTP, REST, JSON)
- Efficient & cost-effective
- Work well with SDKs in many programming languages
- Customers can create fault-tolerant apps by supporting retries/idempotency
- Sustainable & versionable via clear API contracts with 2 requirements:
  - Customer workloads must never break due to a service change
  - Customers can adopt a version without requiring code changes

Technology and software is constantly changing and evolving, and as such, this is intended to be a living document. [Open an issue](https://github.com/microsoft/api-guidelines/issues/new/choose) to suggest a change or propose a new idea. 

> NOTE: For an existing GA'd service, don't change/break its existing API; instead, leverage these concepts for future APIs while prioritizing consistency within your existing service.

### Prescriptive Guidance  
This document offers prescriptive guidance labeled as follows:

:white_check_mark: **DO** adopt this guideline or follow this pattern. If you feel you need an exception, contact the Azure HTTP/REST Stewardship Board[TODO: mail link? - not for public people] <b>prior</b> to implementation.

:ballot_box_with_check: **YOU SHOULD** strongly consider this guideline. If not following this advice, you MUST disclose your reason during the Azure HTTP/REST Stewardship Board review.

:heavy_check_mark: **YOU MAY** consider this guideline if appropriate to your situation. No notification to the Azure HTTP/REST Stewardship Board is required.

:warning: **YOU SHOULD NOT** strongly consider avoiding the described pattern. If not following this advice, you MUST disclose your reason during the Azure HTTP/REST Stewardship Board review.

:no_entry: **DO NOT** follow this pattern. If you feel you need an exception, contact the Azure HTTP/REST Stewardship Board <b>prior</b> to implementation.

*If you feel you need an exception, or need clarity based on your situation, please contact the Azure HTTP/REST Stewardship Board <b>prior</b> to release of your API.*

## Advice for New Services 
Great APIs make your service usable to customers. They are intuitive, naturally reflecting and communicating the underlying model and its behavior. They lend themselves easily to client library implementations in multiple programming languages. And they don't "get in the way" of the developer, by remaining stable and predictable, especially over time.

This document provides Microsoft teams building Azure services with a set of guidelines that will help service teams build great APIs. The guidelines can be applied to create an API that is approachable, sustainable, and consistent across the Azure platform. We do this by applying a common set of patterns and web standards to the design and development of the API. For developers, a well defined and constructed API enables them to build fault tolerant applications that are easy to maintain, support, and grow. For Azure service teams, the API is often the source of code generation, and enabling a broad audience of developers across multiple languages.

Azure Service teams should engage the API Stewardship Board early in the development lifecycle for guidance, discussion, and review of their API. In addition, it is good practice to perform a security review, especially if you are concerned about PII leakage, compliance with GDPR, or any other considerations relative to your situation.

Your goal is to create a developer friendly API where:

:white_check_mark: **DO** ensure that customer workloads never break

:white_check_mark: **DO** ensure that customers are able to adopt a new version of service or SDK w/out requiring code changes 

## Azure Management Plane vs Data Plane
> Note: Developing a new service requires the development of at least 1 (management plane) API and potentially one or more additional (data plane) APIs.  When reviewing v1 service APIs, we see common advice provided during the review.
> A **management plane** API is implemented through the Azure Resource Manager (ARM) and is used by subscription administrators.  A **data plane** API is used by developers to implement applications.  Rarely, a subset of operations may be useful to both administrators and users, in which case it should appear in both APIs. Although the best practices and patterns described in this document apply to all REST APIs, they are especially important for **data plane** services because it is the primary interface for developers using your service. The **management plane** APIs may have other preferred practices based on the conventions of the Azure ARM.

### Start with developer experience
A great API starts with a well thought out and designed service. It is extremely difficult to create an elegant API that will work well on top of a service that is poorly designed. It is important that your development team builds some client code using the API. Hold reviews and share what is learend with your team.  Engage with your customers during a preview release.  If during a preview you discover that customers are struggling to use your API, e.g. they don't understand the abstraction layer, take the time to fix your service abstractions. This will benefit the developer and your team. Put yourself in the developer's shoes and think deeply about how they will be using your API and your service.

Think about the code that a customer will write both before and after the REST API call.  What data structures will they need to assemble?  What is the most likely next call?

:white_check_mark: **DO** provide examples in multiple languages

:white_check_mark: **DO** include at least one dynamically typed language (for example, Python or JavaScript) and one statically typed language (for example, Java or C#).


### Focus on hero scenarios
It is important to realize that writing an API is, in many cases, the easist part of providing a delightful developer experience. There are a large number of downstream activities for each API, e.g. testing, documenation, and creation of client libraries and examples.Focusing on hero scenarios reduces development, support, and maintenace costs; enables teams to align and reach consensus faster; and accelerates the time to delivery. A tell tale sign of a service that has not focused on hero scenarios is "API drift," where endpoints are inconsistent, incomplete, or juxtaposed to one another. Service teams:    

:white_check_mark: **DO** define "hero scenarios" first, then the operations required, & then design the API

:white_check_mark: **DO** provide example code that demonstrates their "Hero Scenarios."

:no_entry: **DO NOT** add APIs for speculative features customers might want 

### Start with your API definition
Understanding how your service will be used and defining its model and interaction patterns--its API--should be one of the earliest activities a service team undertakes. It should be reflect the naming decisions and make it easy for developers to implement your hero scenarios.  
:white_check_mark: **DO** provide an [OpenAPI Definition] (with [autorest extensions](https://github.com/Azure/autorest/blob/master/docs/extensions/readme.md)) that describes the service. The OpenAPI Specification is a key element of the Azure SDK plan and essential to improving the documentation, usability and discoverability of services.

:ballot_box_with_check: **YOU SHOULD** describe their services using ADL *[LINK TO ADL HERE]*.

:ballot_box_with_check: **YOU SHOULD** use ADL to generate the required OpenAPI Definition.

### Use previews to iterate 
 Before releasing your API plan to invest significant design effort, get customer feedback, & iterate through multiple preview releases. This is especially important for V1 as it establishes the abstractions and patterns that developers will use to interact with your service.

:ballot_box_with_check: **YOU SHOULD**  write and test hypotheses about how your customers will use the API. :ballot_box_with_check: **YOU SHOULD**  release and evaluate a minimum of 2 preview versions prior to the first GA release.  
:ballot_box_with_check: **YOU SHOULD**  identify key scenarios or design decisions in your API that you want to test with customers, and ask customers for feedback and to share relevant code samples. 
:ballot_box_with_check: **YOU SHOULD**  consider doing a *code with* exercise in which you actively develop with the customer, observing and learning from their API usage.
:ballot_box_with_check: **YOU SHOULD**  capture what you have learned during the preview stage and share these findings with your team and with the API Stewardship Board.

### Avoid surprises
A major inhibitor to adoption and usage is when an API behaves in an unexpected way. Often, these are subtle design decisions that seem benign at the time, but end up introducing significant downstream friction for developers.

:ballot_box_with_check: **YOU SHOULD** avoid polymorphism, especially in the response. An endpoint __SHOULD__ work with a single type to avoid problems during SDK creation.

:ballot_box_with_check: **YOU SHOULD** make [Collections](#Collections) easy to work with. Collections are a common source of review comments. It is important to handle them in a consistent manner within your service.

:ballot_box_with_check: **YOU SHOULD** return a homogeneous collection (single type).  Do not return heterogeneous collections unless there is a really good reason to do so.  If you feel heterogeneous collections are required, discuss the requirement with an API reviewer prior to implementation.

:ballot_box_with_check: **YOU SHOULD** support server-side paging, even if your resource does not currently need paging. This avoids a breaking change when your service expands.

### Design for Resiliency 
As you build out your service and API, there are a number of decisions that can be made up front that add resiliency. Addressing these as early as possible will help you iterate faster and avoid breaking changes.

:ballot_box_with_check: **YOU SHOULD** use extensible enumerations. Extensible enumerations are modeled as strings - expanding an extensible enumeration is not a breaking change. 

:ballot_box_with_check: **YOU SHOULD** implement [conditional requests](https://tools.ietf.org/html/rfc7232) early. This allows you to support concurrency, which tends to be a concern later on.

:ballot_box_with_check: **YOU SHOULD** use wider data types (e.g. 64-bit vs. 32-bit) as they are more future-proof. For example, JavaScript can only support numbers up to 2<sup>53</sup>, so relying on the full width of a 64-bit number should be avoided.

## Building Blocks: HTTP, REST, & JSON
The Microsoft Azure Cloud platform exposes its APIs through the core building blocks of the Internet, namely HTTP, REST, and JSON. This section will provide you with a general understanding of how these technologies should be applied when creating your service.

### HTTP
Azure services will adhere to the HTTP specification, [RFC7231](https://tools.ietf.org/html/rfc7231), as closely possible when presenting their API. This section further refines and constrains how service implementors should apply the constructs defined in the HTTP specification. It is therefore, important that you have a firm understanding of the following concepts:
* [Uniform Resource Locators (URLs)](URLS)
* HTTP Methods
* Headers
* Bodies

#### URLs 
<span style="color:red; font-size:large">TODO: Update this section </span>

A Uniform Resource Locator (URL) is how developers will access the resources of your service. Ultimately, URLs will be how developers begin to form a cognitive model of your service. These are so central to the developer experience that careful consideration should be given when devising your URL structure.

## URLs 
A Uniform Resource Locator (URL) is how developers access your service's resources. The structure of the URL is critically important as it describes the service's cognitive model:

In addition to the [URL structure guidance](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#71-url-structure) in the Microsoft REST API guidelines, Azure has specific guidance about service exposure for multi-tenant services. Specifically:

:white_check_mark: **DO** expose their service to developers via the following URL pattern:

```text
https://<service>.<cloud>/<tenant>/<service-root>/<resource-collection>/<resource-id>/
```

Where:

* **service**: name of the service (ex: blobstore, servicebus, directory, or management)
* **cloud**: cloud domain name (see Azure CLI's "az cloud list")

   | Cloud         | Domain            |
   | ------------- | -----             |
   | Public        | azure.net         |
   | China         | chinacloudapi.cn  |
   | US Government | usgovcloudapi.net |
   | German        | cloudap.de        |

* **tenant**: globally unique ID of container representing tenant isolation, billing, enforced quotas, lifetime of containees (ex: subscription UUID)

* **service-root**: service-specific path (ex: blobcontainer, myqueue)

A service URL must be case-sensitive (except for scheme/host). If case doesn't match what you expect, the request __MUST__ fail with the appropriate HTTP return code.

When returning information in a Response, services __MUST__ maintain and respect proper case values.

A well formed Azure service URL would look similar to the following:

<span style="color:red; font-size:large">TODO: Show URL example </span>


:ballot_box_with_check: **YOU SHOULD** keep URLs readable; if possible, avoid UUIDs & %-encoding (ex: Cádiz)

### Direct endpoint URLs

:ballot_box_with_check: **YOU SHOULD** use case-sensitive comparison for <resource-id> 

:heavy_check_mark: **YOU MAY** use case-insensitive comparison for a <resrouce-id> that is a GUID value

---
A direct endpoint URL <b>may also</b> be used for performance/routing:

    https://<tenant>-<service-root>.<service>.<cloud>/...

    Examples: 
    - Request URL: https://blobstore.azure.net/contoso.com/account1/container1/blob2
    - Response ```content-location``` [RFC2557](https://datatracker.ietf.org/doc/html/rfc2557#section-4): https://contoso-dot-com-account1.blobstore.azure.net/container1/blob2
    - GUID format: https://00000000-0000-0000-C000-000000000046-account1.blobstore.azure.net/container1/blob2
---

:white_check_mark: **DO** return URLs in response headers/bodies in a consistent form regardless of the URL used to reach the resource. Either always a GUID for <tenant> or always a single verified domain.

:white_check_mark: **DO** use kebab-casing (preferred) or camel-casing for URL path segments. If the segment refers to a JSON field, use camel casing.

### HTTP Request / Response Pattern
The HTTP Request / Response pattern will dictate much of how your API behaves, for example; POST methods must be idempotent, GET methods may be cached, the If-Modified and etag headers determine your optimistic concurrency strategy. The URL of a service, along with its request / response, establishes the overall contract that developers have with your service. As a service provider, how you manage the overall request / response pattern should be one of the first implementation decisions you will make. For each request / response, the service: 

:ballot_box_with_check: **YOU SHOULD** try to limit your URL's characters to ```0-9  A-Z  a-z  -  .  _  ~```

:heavy_check_mark: **YOU MAY** use these other characters but they will likely require %-encoding: ```/  ?  #  [  ]  @  !  $  &  '  (  )  *  +  ,  ;  =```

### HTTP Methods & Idempotency
Cloud applications embrace failure. Therefore, to enable customers to write fault-tolerant applications, <b>all</b> service operations (including POST) <b>must</b> be idempotent.

---
> Exactly Once Behavior = Client Retries & Service Idempotency
---

:white_check_mark: **DO** ensure that ALL HTTP methods are idempotent.

:warning: **YOU SHOULD NOT** use POST method unless you implement idempotently via an Idempotent-token header (TODO: fix this up). TODO: Say how POST returns the URL of the create resource with 201

:white_check_mark: **DO** adhere to the return codes in the following table when implementing your API:

Method | Description | Response Status Code 
----|----|----
GET | Read the resource | 200-OK 
DELETE | Remove the resource | 204-No Content\; avoid 404-Not Found 
PATCH | Create/Modify the resource with JSON Merge Patch | 200-OK, 201-Created
PUT | Create/Replace the *whole* resource | 200-OK, 201-Created 
 
<span style="color:red; font-size:large">TODO: To get a collection's resources (GET; see the collection section)</span>

**YOU MAY** support caching and optimistic concurrency by returning resources with an etag response header and by supporting the if-match, if-none-match, if-modified-since, and if-unmodified-since request headers.

### HTTP Query Parameters and Header Values
The table below lists the headers most used by Azure services:

Header Key | Applies to | Example
------------ | ------------- | -------------
*authorization* | Request | Bearer eyJ0...Xd6j (Support Azure Active Directory) 
*x-ms-useragent*  |  Request | [see Telemetry](http://TODO:link-goes-here)
traceparent | Request | [see Distributed Tracing]](http://TODO:link-goes-here)
tracecontext | Request | [see Distributed Tracing](http://TODO:link-goes-here)
accept | Request | application/json
if-match | Request | "67ab43" or * (no quotes) (see Conditional Access)
if-none-match | Request | "67ab43" or * (no quotes) [see Conditional Access](http://TODO:link-goes-here)
If-Modified-Since | Request | (RFC1123) [see Optimistic Concurrency](http://TODO:link-goes-here)
If-Unmodified-Since | Request | (RFC1123) [see Optimistic Concurrency](http://TODO:link-goes-here)
date [RFC1123](https://datatracker.ietf.org/doc/html/rfc1123) | Both | Sun, 06 Nov 1994 08:49:37 GMT 
*content-type* | Both | application/merge-patch+json
*content-length* | Both | 1024
*x-ms-request-id* | Response | [see Customer Support](http://TODO:link-goes-here)
etag | Response | "67ab43" [see Conditional Access](http://TODO:link-goes-here)
retry-after | Response | 180 [see Throttling Client Requests]
*x-ms-error-code* | Response | [see Processing a REST Request](http://TODO:link-goes-here)
Last-Modified | Response | (RFC1123) [see Optimistic Concurrency](http://TODO:link-goes-here)

:white_check_mark: **DO** specify headers using kabob-casing.

:white_check_mark: **DO** compare request header names using case-insensitivity

:white_check_mark: **DO** compare request header values using case-sensitivity. Some exceptions exist: user-agent?, accept?, content-type?, RFC1123 dates, guids?.

#### HTTP Methods & Idempotency
Implementing services in an idempotent manner, with an "exactly once" semantic, enables developers to retry requests without the risk of unintended consequences.
:ballot_box_with_check: **YOU SHOULD** properly handle all headers annotated in *italics*. In addition, each request / response header. 

:no_entry: **DO NOT** use "x-" prefix for headers, unless the header already exists in production.

:warning: **YOU SHOULD NOT** using the POST method unless you can guarantee it can be implemented idempotently.
:white_check_mark: **DO** validate all query parameter and request header values. TODO: What to return on failure

:white_check_mark: **DO** return the state of the resource after a PUT, PATCH, or GET operation with a ```200-OK``` or ```201-Created```.

Method | Description | Response Status Code
----|----|----
GET | Read the resource | 200-OK
DELETE | Remove the resource | 204-No Content; avoid 404-Not Found
PATCH | Create/Modify the resource with JSON Merge Patch | 200-OK, 201-Created
PUT | Create/Replace the *whole* resource | 200-OK, 201-Created
:white_check_mark: **DO** return a ```204-No Content``` without a resource for a DELETE operation (even if the URL identifies a resource that does not exist; do not return ```404-Not Found```)
 
Because information in the service URL, as well as the request / response, are strings, there must be a predictable, well-defined scheme to convert strings to their corresponding values.

:white_check_mark: **DO** use the following table when translating strings: 

Data type | Document that string must be
-------- | -------
Boolean  | true / false (all lowercase)
Integer  | -2<sup>53</sup>+1 to +2<sup>53</sup>-1 (limit due to IEEE-754 [RFC8259](https://datatracker.ietf.org/doc/html/rfc8259))
Float    | [IEEE-754 binary64](https://en.wikipedia.org/wiki/Double-precision_floating-point_format) 
String   | (Un)quoted?, max length, legal characters, case-sensitive, multiple delimiter 
UUID     | {}? casing? hyphens? [RFC4122](https://datatracker.ietf.org/doc/html/rfc4122)
Date/Time (Header) | [RFC1123](https://datatracker.ietf.org/doc/html/rfc1123)
Date/Time (Query parameter) | YYYY-MM-DDTHH:mm:ss.sssZ [RFC3339](https://datatracker.ietf.org/doc/html/rfc3339) 
Byte array | Base-64 encoded, max length 

<span style="color:red; font-size:large">TODO: Expand the explanation for numbers. </span>

<span style="color:red; font-size:large">TODO: Fix the links. </span>

#### Additional References
* [StackOverflow - Difference between http parameters and http headers](https://stackoverflow.com/questions/40492782)
* [Standard HTTP Headers](https://httpwg.org/specs/rfc7231.html#header.field.registration)
* [Why isn't HTTP PUT allowed to do partial updates in a REST API?](https://stackoverflow.com/questions/19732423/why-isnt-http-put-allowed-to-do-partial-updates-in-a-rest-api)

### REST
REST is an architectural style with broad reach that emphasizes scalability, generality, independent deployment, reduced latency via caching, and security. When applying REST to your API, you will define your service’s resources as a collections of items. These are typically the nouns you use in the vocabulary of your service. Your service's [URLs](#URLS) determine the hierarchical path developers use to retrieve and update the state of your resource. Note, it's important to model resource state, not to behavior. There are patterns, later in these guidelines, that describe how to invoke behavior on your service.
<span style="color:red; font-size:large">TODO: Add link to behavior section </span>

When designing your service, it is important to optimize for the developer using your API.

:white_check_mark: **DO** focus heavily on clear & consistent naming

:white_check_mark: **DO** ensure your resource paths make sense

:white_check_mark: **DO** simplify call with few required query parameters & JSON fields

:white_check_mark: **DO** establish clear contracts for string values

:white_check_mark: **DO** use proper response codes/payloads so customer can self-fix

#### JSON Resource Schema & Field Mutability
For a given URL path, the JSON schema (data type) should be the same for PATCH, PUT, GET, DELETE, and GETting collection items. This allows one SDK type for input/output operations and enables the response to be passed back in request. While not explicitly defined in JSON, each field in your JSON schema should have an associated mutability rule. <!--Tools like ADL do allow annotation of mutability, enabling more sophisticated code generation of client libraries.-->
:white_check_mark: **DO** use the same JSON schema for PUT request/response, PATCH request/response, GET response, and POST response. This allows one SDK type for input/output operations and enables the response to be passed back in request.

---
<span style="color:red;">Jeff: What to say about required fields - it not really a REST thing 
TODO: Some fields may be marked as required indicating that their value must alway be sent <b>and</b> is always returned. </span>


> NOTE: A service is <b>not allowed</b> to introduce new required fields or remove any required fields in newer versions of the service.
For PATCH, the their must be a similar JSON schema with no required fields nullable.
---

---
This is also not really a rest thing; more of a service implementation thing
While not explicitly defined in JSON, each field in your JSON schema should have an associated mutability rule. REMOVE?: Tools like ADL do allow annotation of mutability, enabling more sophisticated code generation of client libraries. 

:white_check_mark: **DO** create a model of your data types. For each field, apply one of the following rules:

Field Mutability | Service Request's behavior for this field
----| ----
**Create** | Service honors field only when creating a resource. Minimize create-only fields so customers don't have to delete & re-create the resource.
**Update** | Service honors field when creating or updating a resource
**Read**   | Service fails request (or accept if they match what's in the resource);returns these fields in a response

---

#### General guidelines
The following are general guidelines when using REST:

:white_check_mark: **DO** serve GET for resource retrieval and send JSON in the response body.

:white_check_mark: **DO** create and update resources using PATCH [RFC5789] with JSON Merge Patch [(RFC7396)](https://datatracker.ietf.org/doc/html/rfc7396) request body.

:white_check_mark: **DO** use PUT with JSON for wholesale create/update operations.
> NOTE: If a v1 client PUTs a resource; any fields introduced in V2+ should be reset to their default values (the equivalent tof DELETE followed by PUT).

:white_check_mark: **DO** use DELETE to remove a resource.

TODO: If we keep this NOTE, then it is about IDs, not about DELETE: NOTE: Ids are "Customer Content" & Azure allows their use.

:white_check_mark: **DO** make fields simple and maintain a shallow hierarchy.

:white_check_mark: **DO** treat JSON field names with case-sensitivity.

:white_check_mark: **DO** treat JSON field values with case-sensitivity. There may be some exceptions but avoid if at all possible.

:no_entry: **DO** fail an operation with ```400-Bad Request``` if the request JSON body is improperly-formed JSON.

:no_entry: **DO** fail an operation with ```412-Unprocessable Entity``` if any JSON field name or value is not fully understood by the specific version of the service.

:no_entry: **DO NOT** return secret fields via GET. For example, do not return adminPassword in JSON.

:no_entry: **DO NOT** add computed output values if the computed value can be calculated from other information in the payload. It unnecessarily expands the payload.

:heavy_check_mark: **YOU MAY** return secret fields via POST **if absolutely necessary**. 


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

:white_check_mark: **DO** return x-ms-error-code header with string

:white_check_mark: **DO** ensure your service returns the error response body (TODO: show it here)

### JSON
Services, and the clients that access them, may be written in multiple languages. To ensure interoperability, JSON establishes the "lowest common denominator" type system, which is always sent over the wire as UTF-8 bytes. This system is very simple and consists of three types:
* **Boolean:**	true/false
* **Number:** signed floating point (IEEE-754 binary64; int range: -2<sup>53</sup>+1 to +2<sup>53</sup>-1)
* **String:** used for everything else

:white_check_mark: **DO** use integers within the acceptable range of JSON number.

#### String Contracts
When using strings, you must establish, and adhere to, a well defined contract for the format. For example, you should be cognizant of attributes like maximum length, legal characters, case-sensitivity, etc. Where possible, use standard formats, e.g. RFC3339 for date/time.

:white_check_mark: **DO** ensure that information exchanged between your service and any client is "round-trippable." 
:white_check_mark: **DO** use [RFC3339] for date/time.
:white_check_mark: **DO** use [RFC4122] for UUIDs.

##### Composite types
JSON also supports composing strings into higher order constructs, for example:
* **Object**:	{ "name" : value, … }
* **Array**:	[ value, … ]

:warning: **YOU SHOULD NOT** use JSON Arrays, e.g. [ value, … ]. Arrays are very difficult and inefficient to work with, especially with updates when using ```JSON Merge Patch```, as the entire array needs to be read prior to any operation being applied to it.

:ballot_box_with_check: **YOU SHOULD** use maps instead of arrays.

#### Enums & SDKs (Client libraries)

It is common for strings to have an explicit set of values. These are often reflected in the OpenAPI specification as enumerations. These are extremely useful for developer tooling, e.g. code completion, and client library generation. However, your services will have client libraries in many different programming languages. And because enumerations are handled differently depending on the language, this can lead to significant interoperability issues.

To address these issues, Microsoft's tooling uses the concept of an "extensible enum," which effectively treats all enumerations as strings. In addition, "extensible enums" indicate to client libraries that the list of values is only a *partial* list. This enables the set of values to grow over time while ensuring stability in client libraries.

:white_check_mark: **DO** use "extensible enums"

:ballot_box_with_check: **DO** document to customers that new values may appear in the future so that customers write their code today expecting these new values tomorrow.

:no_entry: **DO NOT** send "enum integers" over the wire.

:no_entry: **DO NOT** remove values from your enumeration list. This will likely result in a breaking change to client libraries & customers.

#### Discriminate polymorphic types 
While polymorphism is a powerful concept in programming languages, returing "polymorphic JSON" as part of an API introduces significant complexity for implementors of client libraries and developers, especially as new versions of your service are introduced. For example, consider a service where V1 introduces two shapes, Retangles and Circles. They could be represented in JSON as follows:

**Rectangle**
```json
{"kind": "rectangle", 
"x": 100, "y": 50, "width": 10, "length": 24, "fillColor": "Red", "lineColor": "White", 
"subscription": {"expiration": "2024"    "kind": "free"}}
```

**Circle**
 ```json
 {"kind": "circle",
"x": 100, "y": 50, "radius": 10, "fillColor": "Green", "lineColor": "Black", 
"subscription": { "expiration": "2024", "kind": "paid", "invoice": "123456"}}``
```

The first issue is that developers writing code against this JSON string contract will have a very difficult time, especially in typed languages. It will be impossible to determine what the actual type is during development, minimizing the effectiveness of tooling. At runtime, developers will have to parse the JSON, interpret the "kind" value, and *then* cast to the proper sub-class.  

Overall, this is a very brittle design that leads to a poor developer experience, especially over time. Consider the scenario when a new shape is introduced in V2 of the API. Existing client libraries that work with V1 will have no concept of this new shape and, when receiving an unknown shape, fail.

:warning: **YOU SHOULD NOT** use polymorphic types. Instead, return discriminate types. 

## Common API Patterns

### Performing an Action

### Collections

### API Versioning

Versioning is a powerful technique that helps you provide consistent and stable APIs to developers that use your service. The Microsoft REST API guidelines offer different options on how to specify an API version and guidance on what constitutes a breaking change. However, this section refines that guidance in order to drive greater consistency across the Azure services and stack, as well as the developer tooling, and client libraries. 

:white_check_mark: **DO** use explicit versioning for your service API.

:white_check_mark: **DO** follow the standard [_Azure Global Retirements and Breaking Changes_][7] when retiring your API.

#### Specifying the version in Azure
The Microsoft REST API guidelines give two options for how services and clients communicate the version: a url segment and a query parameter. For consistency across the Azure portfolio, you must specify the version using a query parameter.

Examples: 
```text
GET https://blobstore.azure.com/foo.com/acct1/c1/blob2?api-version=1.0
PUT https://blobstore.azure.com/foo.com/acct1/c1/b2?api-version=2014-12-07
POST https://blobstore.azure.com/foo.com/acct1/c1/b2?api-version=2015-12-07
```

#### API Changes that require a version change

There are three groups of changes that may happen to an API.

1. Changes made to an **EXISTING** API version due to security or compliance reasons.  We shall refer to these types of changes as _Compliance changes_.
2. Changes made to an API that may cause a client making the API call to fail, such as removal of an endpoint or property or changing the format of the body.  We refer to these types of changes as _Breaking changes_.
3. Additive changes made to an API that do not cause a client making the API call to fail, such as the addition of a new optional property or a new endpoint.  We refer to these types of changes as _Evolutionary changes_.

With the exception of _Compliance changes_ (which are extremely rare), Azure services must update the version number of their API whenever there is a change to the API, _no matter how small_.  Customers will "lock the API version" so that their code does not fail when the service introduces new features and will rely on the fact that your API version is a contract with the services that will never change.

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


:ballot_box_with_check: **YOU SHOULD** use a date oriented semantic as the version numbering scheme of your API, e.g. `2021-05-01-preview.1`.

:white_check_mark: **DO** increment the version of your API for _Compliance changes_.

:white_check_mark: **DO** review any breaking changes with the Azure REST API stewardship board prior to seecing approval through the [Azure Global Breaking Change Policy][7]. 

:white_check_mark: **DO** follow the [Azure Global Retirement Policy][7] for any deprecated APIs.  

:white_check_mark: **DO** increment the major version (if using SemVer) for any breaking changes.

:white_check_mark: **DO** increment the version bump for _evolutionary changes. If the service is using SemVer for versioning, evolutionary changes constitute a minor version change.

:ballot_box_with_check: **YOU SHOULD** review any _evolutionary change_ with the Azure API stewardship board.


#### Changing the API without changing the version

Because the API version represents a contract that a developer can rely on when generating SDKs to communicate with the service, there are a limited set of situations where changing the API is permissable without a version bump.  The only changes universally allowed:

1. Adding a new (optional) value to an extensible enum.

An extensible enum is (in essence) a string.  The values of the extensible enum drive intellisense and documentation, but the values are not considered exhaustive.

If a service is **ONLY** available in the Azure public cloud, then an additional situation can be used to add functionality without changing the version:

1. Adding a new (optional) query parameter to adjust the output (for example, adding filter options to a list operation).
2. Adding optional computed read-only output values that are generated based on the new (optional) query parameters.

For example, let's say an image service wants to add bounding-box information to the output of an operation.  The service can add a new query parameter `includeBoundingBox=true` and then include the bounding box information within the output only when the new query parameter is specified.  A version bump is recommended, but not required.  

If not changing the API version, the service **MUST** update all data centers before the new query parameter is advertised to customers.

:no_entry: **DO NOT** use this mechanism to get around the version bump.  Adding such query parameters results in sub-optimal API designs and should only be used for exceptional circumstances.

This functionality is only available for single cloud deployments because the API version specifies the contract with the developer. Consider, for example, if such a functionality was included in Azure public cloud and not a sovereign cloud. A developer creating an SDK based on this functionality may see the application work in one cloud but fail when targeting the other despite using the same API version in both cases.  For the purposes of this situation, "other clouds" includes Azure Stack and other deployment mechanisms such as containers for on-premise usage.


All situations where the API definition is changed (irrespective of whether a version change happens or not)
 **MUST** be reviewed by the Azure REST API Review Board before release.  

#### Preview Versions

Preview versions of the API can be indicated by adding the suffix `-preview.X` to the end of the API version, where `X` is an incrementing integer.  For example:

* `2021-05-01-preview.1`
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

#### Group versioning in Azure and Azure Stack

Azure Stack allows customers and hosters to deploy their own small versions of Azure and upgrade it at a different pace than Azure. In order to make it possible to write an application or SDK that targets both Azure and Azure Stack, additional versioning policy is necessary. Contact the Azure Stack team for further guidance.

#### Version discovery

Simpler clients may be hardcoded to a single version of a service. Since Azure services offer each version for a well-known period of time, a client that’s regularly maintained can be always operational without further complexity as long as during regular maintenance the client is moved forward to new versions in advance of older ones being retired.

API version discovery is needed when either a given hosted service may expose a different API version to different clients (e.g. latest API version only available in certain regions or to certain tenants) or the service itself may exist in different instances (e.g. a service that may be run on Azure or hosted on-premises). In both of those cases clients may get ahead of services in the API version they use. In might also be possible for a client version to ship ahead of its corresponding service update, leading to the same situation. Lastly, version discovery is useful for clients that want to warn operators that an API they depend on may expire soon.

:white_check_mark: **DO**  support API version discovery, including 

1. Support HTTP `OPTIONS` requests against all resources, including the root URL for a given tenant or the global root if no tenant identity is tracked or not a multi-tenant service
2. Include the `api-supported-versions` header, containing a comma-separated list of versions conforming to the Azure versioning scheme. This list must include all group versions as well as all major-minor versions supported by the target resource. For cases where no specific version applies (e.g. sometimes the root resource), the list still must contain the group versions supported by the service.
3. If a given service supports versions of the API that are known to be planned for deprecation in a year or less, it must include those versions (group and major.minor) in the `api-deprecated-versions` header.
4. For services that do rolling updates where there is a point in time where some front-ends are ahead of others version-wise, all front-ends **MUST** report the previous version as the latest version until the rolling update covers all instances and only then switch over to reporting the new latest version. This ensures that clients will not detect a version and then get load-balanced into a front-end that does not support it yet.

:ballot_box_with_check: **YOU SHOULD** support the following for version discovery:

1. In addition to the functionality described here, services should support HTTP `OPTIONS` requests for other purposes such as further discovery, CORS, etc.
2. Services should allow unauthenticated HTTP `OPTIONS` requests. When doing so, authors need to consider whether HTTP `OPTIONS` requests against non-existing resources result in 404s and whether that is leaking sensitive information. Certain scenarios, such as support for CORS pre-flight requests, require allowing unauthenticated HTTP `OPTIONS` requests.
3. If using OData and addressing an expanded resource, the HTTP `OPTIONS` request should return the group versions that are supported across the expanded set.

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

#### Retiring pre-release and beta APIs

Pre-release and beta APIs are not covered by the Azure Global Retirement and Deprecation Policy. Each team providing a preview API should communicate to customers what the policy is going to be for support and deprecation, even if that policy is “we may remove this at any time”. The Azure REST API Guidelines cover pre-release API versions. To summarize that section, they should be marked with a version tag like `2013-03-21-Preview`. Despite their unstable nature and the fact they are not officially supported, **customers have suffered downtime when we have deprecated preview APIs**. 

:ballot_box_with_check: **YOU SHOULD** should monitor preview endpoints closely.

:ballot_box_with_check: **YOU SHOULD** consider following the normal deprecation policy. 

##### Additional References

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


### Long Running Operations & Jobs

The Microsoft REST API guidelines for Long Running Operations are an updated, clarified and simplified version of the Asynchronous Operations guidelines from the 2.1 version of the Azure API guidelines. Unfortunately, to generalize to the whole of Microsoft and not just Azure, the HEADER used in the operation was renamed from `Azure-AsyncOperation` to `Operation-Location`. 

:white_check_mark: **DO** support both `Azure-AsyncOperation` and `Operation-Location` HEADERS, even though they are redundant so that existing SDKs and clients will continue to operate. 

:white_check_mark: **DO** return the same value for **both** headers.

:white_check_mark: **DO** look for **both** HEADERS in client code, preferring the `Operation-Location` version. 




### Distributed Tracing & Service Telemetry
* Distributed Tracing  
* Service Telemetry 
* How to collect client side telemetry 


### Bring your own storage
* Getting data into your service
* Working with blobs

### Optimistic concurrency
* Conditional Access 
* Optimistic Concurrency 
* Etags 


### Getting data into your service 
* Bring your own storage 
* Working with blobs 


## Final Thoughts / Summary
* Careful consideration up front
* Long term decisions that are often codified in SDKs, CODE, etc.
* Reach out and engage the stewardship team!
