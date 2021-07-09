## Considerations for Service Design 
Great APIs make your service usable to customers. They are intuitive, naturally reflecting and communicating the underlying model and its behavior. They lend themselves easily to client library implementations in multiple programming languages. And they don't "get in the way" of the developer, by remaining stable and predictable, <i>especially over time</i>.

This document provides Microsoft teams building Azure services with a set of guidelines that  help service teams build great APIs. The guidelines create APIs that are approachable, sustainable, and consistent across the Azure platform. We do this by applying a common set of patterns and web standards to the design and development of the API. For developers, a well defined and constructed API enables them to build fault-tolerant applications that are easy to maintain, support, and grow. For Azure service teams, the API is often the source of code generation enabling a broad audience of developers across multiple languages.

Azure Service teams should engage the  Azure HTTP/REST Stewardship Board early in the development lifecycle for guidance, discussion, and review of their API. In addition, it is good practice to perform a security review, especially if you are concerned about PII leakage, compliance with GDPR, or any other considerations relative to your situation.

Your goal is to create a developer friendly API where:

> :white_check_mark: **DO** ensure that customer workloads never break
> 
> :white_check_mark: **DO** ensure that customers are able to adopt a new version of service or SDK client library <b>without requiring code changes</b> 

### Azure Management Plane vs Data Plane
*Note: Developing a new service requires the development of at least 1 (management plane) API and potentially one or more additional (data plane) APIs.  When reviewing v1 service APIs, we see common advice provided during the review.*

A **management plane** API is implemented through the Azure Resource Manager (ARM) and is used by subscription administrators. A **data plane** API is used by developers to implement applications. Occasionally, some operations are useful to both administrators and developers. In this case, the operation can appear in both APIs. Although, best practices and patterns described in this document apply to all HTTP/REST APIs, they are especially important for **data plane** services because it is the primary interface for developers using your service. The **management plane** APIs may have other preferred practices based on [the conventions of the Azure ARM](https://github.com/Azure/azure-resource-manager-rpc).

### Start with the Developer Experience
A great API starts with a well thought out and designed service. Your service should define simple/understandable abstractions with each given a clear name that you use consistently throughout your API and documentation. There must also be an unambiguous relationship between these abstractions.

It is extremely difficult to create an elegant API that works well on top of a poorly designed service; the service team and customers will live with this pain for years to come. So, the service team should empathize with customers by:
 - Building apps that consume the API
 - Hold reviews and share what is learned with your team
 - Get customer feedback from API previews
 - Thinking about the code that a customer writes both before and after an HTTP operation
 - Initializing and reading from the data structures your service requires
 - Thinking about which errors are recoverable at runtime as opposed to indicating a bug in the customer code that must be fixed

The whole purpose of a preview to address feedback by improving abstractions, naming, relationships, API operations, and so on. It is OK to make breaking changes during a preview to improve the experience now so that it is sustainable long term.

### Focus on Hero Scenarios
It is important to realize that writing an API is, in many cases, the easiest part of providing a delightful developer experience. There are a large number of downstream activities for each API, e.g. testing, documentation, client libraries, examples, blog posts, videos, and supporting customers in perpetuity. In fact, implementing an API is of miniscule cost compared to all the other downstream activities. 

*For this reason, it is <b>much better</b> to ship with fewer features and only add new features over time as required by customers.*

Focusing on hero scenarios reduces development, support, and maintenance costs; enables teams to align and reach consensus faster; and accelerates the time to delivery. A telltale sign of a service that has not focused on hero scenarios is "API drift," where endpoints are inconsistent, incomplete, or juxtaposed to one another. 

> :white_check_mark: **DO** define "hero scenarios" first including abstractions, naming, relationships, and then define the API describing the operations required
> 
> :white_check_mark: **DO** provide example code demonstrating the "Hero Scenarios"
> 
> :no_entry: **DO NOT** proactively add APIs for speculative features customers might want 

> :white_check_mark: **DO** consider how your abstractions will be represented in different high-level languages.

> :white_check_mark: **DO** develop code examples in at least one dynamically typed language (for example, Python or JavaScript) and one statically typed language (for example, Java or C#) to illustrate your abstractions and high-level language representations.

### Start with your API Definition
Understanding how your service is used and defining its model and interaction patterns--its API--should be one of the earliest activities a service team undertakes. It reflects the abstractions & naming decisions and makes it easy for developers to implement the hero scenarios.  

> :white_check_mark: **DO** provide an [OpenAPI Definition](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/2.0.md) (with [autorest extensions](https://github.com/Azure/autorest/blob/master/docs/extensions/readme.md)) describing the service. The OpenAPI definition is a key element of the Azure SDK plan and is essential for documentation, usability and discoverability of services.

### Use Previews to Iterate 
 Before releasing your API plan to invest significant design effort, get customer feedback, & iterate through multiple preview releases. This is especially important for V1 as it establishes the abstractions and patterns that developers will use to interact with your service.

> :ballot_box_with_check: **YOU SHOULD**  write and test hypotheses about how your customers will use the API.
> 
> :ballot_box_with_check: **YOU SHOULD**  release and evaluate a minimum of 2 preview versions prior to the first GA release.
> 
> :ballot_box_with_check: **YOU SHOULD**  identify key scenarios or design decisions in your API that you want to test with customers, and ask customers for feedback and to share relevant code samples.
> 
> :ballot_box_with_check: **YOU SHOULD**  consider doing a *code with* exercise in which you actively develop with the customer, observing and learning from their API usage.
> 
> :ballot_box_with_check: **YOU SHOULD**  capture what you have learned during the preview stage and share these findings with your team and with the API Stewardship Board.

### Avoid Surprises
A major inhibitor to adoption and usage is when an API behaves in an unexpected way. Often, these are subtle design decisions that seem benign at the time, but end up introducing significant downstream friction for developers.

> :ballot_box_with_check: **YOU SHOULD** avoid polymorphism, especially in the response. An endpoint __SHOULD__ work with a single type to avoid problems during SDK creation.
>
> :ballot_box_with_check: **YOU SHOULD** make [Collections](./Guidelines.md#collections) easy to work with. Collections are a common source of review comments. It is important to handle them in a consistent manner within your service.
>
> :ballot_box_with_check: **YOU SHOULD** return a homogeneous collection (single type).  Do not return heterogeneous collections unless there is a really good reason to do so.  If you feel heterogeneous collections are required, discuss the requirement with an API reviewer prior to implementation.
>
> :ballot_box_with_check: **YOU SHOULD** support server-side paging, even if your resource does not currently need paging. This avoids a breaking change when your service expands.

### Design for Change Resiliency 
As you build out your service and API, there are a number of decisions that can be made up front that add resiliency to client implementations. Addressing these as early as possible will help you iterate faster and avoid breaking changes.

> :ballot_box_with_check: **YOU SHOULD** use extensible enumerations. Extensible enumerations are modeled as strings - expanding an extensible enumeration is not a breaking change. 
>
> :ballot_box_with_check: **YOU SHOULD** implement [conditional requests](https://tools.ietf.org/html/rfc7232) early. This allows you to support concurrency, which tends to be a concern later on.
>
> :ballot_box_with_check: **YOU SHOULD** use wider data types (e.g. 64-bit vs. 32-bit) as they are more future-proof. For example, JavaScript can only support integers up to 2<sup>53</sup>, so relying on the full width of a 64-bit integer should be avoided.

## Getting Help: The Azure REST API Stewardship Board
The Azure REST API Stewardship board is a collection of dedicated architects that are passionate about helping Azure service teams build interfaces that are intuitive, maintainable, consistent, and most importantly, delight our customers. Because APIs affect nearly all downstream decisions, you are encouraged to reach out to the Stewardship board early in the development process. These architects will work with you to apply these guidelines and identify any hidden pitfalls in your design.
 
### Typical Review Session   
When engaging with the API REST Stewardship board, your working sessions will generally focus on three areas:
* Correctness - Your service should leverage the proper HTTP verbs, return codes, and respect the core constructs of a REST API, e.g. idempotency, that are standard throughout the industry. 
* Consistency - Your services should look and behave as though they are natural part of the Azure platform.
* Well formed - Do your services adhere to REST and Azure standards, e.g. proper return codes, use of headers. 
* Durable - Your APIs will grow and change over time and leveraging the common patterns described in this document will help you minimize your tech debt and move fast with confidence. 

It was once said that "all roads lead to Rome." For cloud services, the equivalent might be that "all 'roads' start with your API." That could not be more true than at Microsoft, where client libraries, documentation, and many other artifacts all originate from the fundamental way you choose to expose your service. With careful consideration at the outset of your development effort, the architectural stewardship of the API board, and the thoughtful application of these guidelines, you will be able to produce a consistent, well formed API that will delight our customers.