# Microsoft Azure REST API Guidelines

<!-- cspell:ignore autorest, BYOS, etag, idempotency, maxpagesize, innererror, trippable, nextlink, condreq, etags -->
<!-- markdownlint-disable MD033 MD049 MD055 -->

<!--
Note to contributors: All guidelines now have an anchor tag to allow cross-referencing from associated tooling.
The anchor tags within a section using a common prefix to ensure uniqueness with anchor tags in other sections.
Please ensure that you add an anchor tag to any new guidelines that you add and maintain the naming convention.
-->

## History

<details>
  <summary>Expand change history</summary>

| Date        | Notes                                                          |
| ----------- | -------------------------------------------------------------- |
| 2025-Mar-28 | Added guidelines about JSON ID and null values                 |
| 2024-Mar-17 | Updated LRO guidelines                                         |
| 2024-Jan-17 | Added guidelines on returning string offsets & lengths         |
| 2023-May-12 | Explain service response for missing/unsupported `api-version` |
| 2023-Apr-21 | Update/clarify guidelines on POST method repeatability         |
| 2023-Apr-07 | Update/clarify guidelines on polymorphism                      |
| 2022-Sep-07 | Updated URL guidelines for DNS Done Right                      |
| 2022-Jul-15 | Update guidance on long-running operations                     |
| 2022-May-11 | Drop guidance on version discovery                             |
| 2022-Mar-29 | Add guidelines about using durations                           |
| 2022-Mar-25 | Update guideline for date values in headers to follow RFC 7231 |
| 2022-Feb-01 | Updated error guidance                                         |
| 2021-Sep-11 | Add long-running operations guidance                           |
| 2021-Aug-06 | Updated Azure REST Guidelines per Azure API Stewardship Board. |
| 2020-Jul-31 | Added service advice for initial versions                      |
| 2020-Mar-31 | 1st public release of the Azure REST API Guidelines            |

</details>

## Introduction

These guidelines apply to Azure service teams implementing _data plane_ APIs. They offer prescriptive guidance that Azure service teams MUST follow ensuring that customers have a great experience by designing APIs meeting these goals:
- Developer friendly via consistent patterns & web standards (HTTP, REST, JSON)
- Efficient & cost-effective
- Work well with SDKs in many programming languages
- Customers can create fault-tolerant apps by supporting retries/idempotency/optimistic concurrency
- Sustainable & versionable via clear API contracts with 2 requirements:
  1. Customer workloads must never break due to a service change
  2. Customers can adopt a version without requiring code changes

Technology and software is constantly changing and evolving, and as such, this is intended to be a living document. [Open an issue](https://github.com/microsoft/api-guidelines/issues/new/choose) to suggest a change or propose a new idea. Please read the [Considerations for Service Design](./ConsiderationsForServiceDesign.md) for an introduction to the topic of API design for Azure services. *For an existing GA'd service, don't change/break its existing API; instead, leverage these concepts for future APIs while prioritizing consistency within your existing service.*

*Note: If you are creating a management plane (ARM) API, please refer to the [Azure Resource Manager Resource Provider Contract](https://github.com/cloud-and-ai-microsoft/resource-provider-contract).*

### Prescriptive Guidance
This document offers prescriptive guidance labeled as follows:

:white_check_mark: **DO** adopt this pattern. If you feel you need an exception, contact the Azure HTTP/REST Stewardship Board **prior** to implementation.

:ballot_box_with_check: **YOU SHOULD** adopt this pattern. If not following this advice, you MUST disclose your reason during the Azure HTTP/REST Stewardship Board review.

:heavy_check_mark: **YOU MAY** consider this pattern if appropriate to your situation. No notification to the Azure HTTP/REST Stewardship Board is required.

:warning: **YOU SHOULD NOT** adopt this pattern. If not following this advice, you MUST disclose your reason during the Azure HTTP/REST Stewardship Board review.

:no_entry: **DO NOT** adopt this pattern. If you feel you need an exception, contact the Azure HTTP/REST Stewardship Board **prior** to implementation.

*If you feel you need an exception, or need clarity based on your situation, please contact the Azure HTTP/REST Stewardship Board **prior** to release of your API.*

## Building Blocks: HTTP, REST, & JSON
The Microsoft Azure Cloud platform exposes its APIs through the core building blocks of the Internet; namely HTTP, REST, and JSON. This section provides you with a general understanding of how these technologies should be applied when creating your service.

<a href="#http" name="http"></a>
### HTTP
Azure services must adhere to the HTTP specification, [RFC 7231](https://tools.ietf.org/html/rfc7231). This section further refines and constrains how service implementors should apply the constructs defined in the HTTP specification. It is therefore, important that you have a firm understanding of the following concepts:

- [Uniform Resource Locators (URLs)](#uniform-resource-locators-urls)
- [HTTP Request / Response Pattern](#http-request--response-pattern)
- [HTTP Query Parameters and Header Values](#http-query-parameters-and-header-values)

#### Uniform Resource Locators (URLs)

A Uniform Resource Locator (URL) is how developers access the resources of your service. Ultimately, URLs are how developers form a cognitive model of your service's resources.

<a href="#http-url-pattern" name="http-url-pattern">:white_check_mark:</a> **DO** use this URL pattern:
```text
https://<tenant>.<region>.<service>.<cloud>/<service-root>/<resource-collection>/<resource-id>
```

Where:
 | Field | Description
 | - | - |
 | tenant | Regionally-unique ID representing a tenant (used for isolation, billing, quota enforcement, lifetime of resources, etc.)
 | region | Identifies the tenant's selected region. This region string MUST match one of the strings in the "Name" column returned from running this Azure CLI's "az account list-locations -o table"
 | service | Name of the service (ex: blobstore, servicebus, directory, or management)
 | cloud | Cloud domain name, e.g. `azure.net` (see Azure CLI's "az cloud list")
 | service&#x2011;root | Service-specific path (ex: blobcontainer, myqueue)
 | resource&#x2011;collection | Name of the collection, unabbreviated, pluralized
 | resource&#x2011;id | Id of resource within the resource-collection. This MUST be the raw string/number/guid value with no quoting but properly escaped to fit in a URL segment.

<a href="#http-url-casing" name="http-url-casing">:white_check_mark:</a> **DO** use kebab-casing (preferred) or camel-casing for URL path segments. If the segment refers to a JSON field, use camel casing.

<a href="#http-url-length" name="http-url-length">:white_check_mark:</a> **DO** return `414-URI Too Long` if a URL exceeds 2083 characters

<a href="#http-url-case-sensitivity" name="http-url-case-sensitivity">:white_check_mark:</a> **DO** treat service-defined URL path segments as case-sensitive. If the passed-in case doesn't match what the service expects, the request **MUST** fail with a `404-Not found` HTTP return code.

Some customer-provided path segment values may be compared case-insensitivity if the abstraction they represent is normally compared with case-insensitivity. For example, a UUID path segment of 'c55f6b35-05f6-42da-8321-2af5099bd2a2' should be treated identical to 'C55F6B35-05F6-42DA-8321-2AF5099BD2A2'

<a href="#http-url-return-casing" name="http-url-return-casing">:white_check_mark:</a> **DO** ensure proper casing when returning a URL in an HTTP response header value or inside a JSON response body

<a href="#http-url-allowed-characters" name="http-url-allowed-characters">:white_check_mark:</a> **DO** restrict the characters in service-defined path segments to `0-9  A-Z  a-z  -  .  _  ~`, with `:` allowed only as described below to designate an action operation.

<a href="#http-url-allowed-characters-2" name="http-url-allowed-characters-2">:ballot_box_with_check:</a> **YOU SHOULD** restrict the characters allowed in user-specified path segments (i.e. path parameters values) to `0-9  A-Z  a-z  -  .  _  ~` (do not allow `:`).

<a href="#http-url-should-be-readable" name="http-url-should-be-readable">:ballot_box_with_check:</a> **YOU SHOULD** keep URLs readable; if possible, avoid UUIDs & %-encoding (ex: Cádiz is %-encoded as C%C3%A1diz)

<a href="#http-url-allowed-characters-3" name="http-url-allowed-characters-3">:heavy_check_mark:</a> **YOU MAY** use these other characters in the URL path but they will likely require %-encoding [[RFC 3986](https://datatracker.ietf.org/doc/html/rfc3986#section-2.1)]: `/  ?  #  [  ]  @  !  $  &  '  (  )  *  +  ,  ;  =`

<a href="#http-direct-endpoints" name="http-direct-endpoints">:heavy_check_mark:</a> **YOU MAY** support a direct endpoint URL for performance/routing:
```text
https://<tenant>-<service-root>.<service>.<cloud>/...
```

Examples:
- Request URL: `https://blobstore.azure.net/contoso.com/account1/container1/blob2`
- Response header ([RFC 2557](https://datatracker.ietf.org/doc/html/rfc2557#section-4)): `content-location : https://contoso-dot-com-account1.blobstore.azure.net/container1/blob2`
- GUID format: `https://00000000-0000-0000-C000-000000000046-account1.blobstore.azure.net/container1/blob2`

<a href="#http-url-return-consistent-form" name="http-url-return-consistent-form">:white_check_mark:</a> **DO** return URLs in response headers/bodies in a consistent form regardless of the URL used to reach the resource. Either always a UUID for `<tenant>` or always a single verified domain.

<a href="#http-url-parameter-values" name="http-url-parameter-values">:heavy_check_mark:</a> **YOU MAY** use URLs as values
```text
https://api.contoso.com/items?url=https://resources.contoso.com/shoes/fancy
```

#### HTTP Request / Response Pattern
The HTTP Request / Response pattern dictates how your API behaves. For example: POST methods that create resources must be idempotent, GET method results may be cached, the If-Modified and ETag headers offer optimistic concurrency. The URL of a service, along with its request/response bodies, establishes the overall contract that developers have with your service. As a service provider, how you manage the overall request / response pattern should be one of the first implementation decisions you make.

Cloud applications embrace failure. Therefore, to enable customers to write fault-tolerant applications, _all_ service operations (including POST) **must** be idempotent. Implementing services in an idempotent manner, with an "exactly once" semantic, enables developers to retry requests without the risk of unintended consequences.

##### Exactly Once Behavior = Client Retries & Service Idempotency

<a href="#http-all-methods-idempotent" name="http-all-methods-idempotent">:white_check_mark:</a> **DO** ensure that _all_ HTTP methods are idempotent.

<a href="#http-use-put-or-patch" name="http-use-put-or-patch">:ballot_box_with_check:</a> **YOU SHOULD** use PUT or PATCH to create a resource as these HTTP methods are easy to implement, allow the customer to name their own resource, and are idempotent.

<a href="#http-post-must-be-idempotent" name="http-post-must-be-idempotent">:heavy_check_mark:</a> **YOU MAY** use POST to create a resource but you must make it idempotent and, of course, the response **MUST** return the URL of the created resource with a 201-Created. One way to make POST idempotent is to use the Repeatability-Request-ID & Repeatability-First-Sent headers (See [Repeatability of requests](#repeatability-of-requests)).

##### HTTP Return Codes

<a href="#http-success-status-codes" name="http-success-status-codes">:white_check_mark:</a> **DO** adhere to the return codes in the following table when the method completes synchronously and is successful:

Method | Description | Response Status Code
-------|-------------|---------------------
PATCH  | Create/Modify the resource with JSON Merge Patch | `200-OK`, `201-Created`
PUT    | Create/Replace the _whole_ resource | `200-OK`, `201-Created`
POST   | Create new resource (ID set by service) | `201-Created` with URL of created resource
POST   | Action | `200-OK`
GET    | Read (i.e. list) a resource collection | `200-OK`
GET    | Read the resource | `200-OK`
DELETE | Remove the resource | `204-No Content`\; avoid `404-Not Found`

<a href="#http-lro-status-code" name="http-lro-status-code">:white_check_mark:</a> **DO** return status code `202-Accepted` and follow the guidance in [Long-Running Operations & Jobs](#long-running-operations--jobs) when a PUT, POST, or DELETE method completes asynchronously.

<a href="#http-method-casing" name="http-method-casing">:white_check_mark:</a> **DO** treat method names as case sensitive and should always be in uppercase

<a href="#http-return-resource" name="http-return-resource">:white_check_mark:</a> **DO** return the state of the resource after a PUT, PATCH, POST, or GET operation with a `200-OK` or `201-Created`.

<a href="#http-delete-returns-204" name="http-delete-returns-204">:white_check_mark:</a> **DO** return a `204-No Content` without a resource/body for a DELETE operation (even if the URL identifies a resource that does not exist; do not return `404-Not Found`)

<a href="#http-post-action-returns-200" name="http-post-action-returns-200">:white_check_mark:</a> **DO** return a `200-OK` from a POST Action. Include a body in the response, even if it has not properties, to allow properties to be added in the future if needed.

<a href="#http-return-403-vs-404" name="http-return-403-vs-404">:white_check_mark:</a> **DO** return a `403-Forbidden` when the user does not have access to the resource _unless_ this would leak information about the existence of the resource that should not be revealed for security/privacy reasons, in which case the response should be `404-Not Found`. [Rationale: a `403-Forbidden` is easier to debug for customers, but should not be used if even admitting the existence of things could potentially leak customer secrets.]

<a href="#http-support-optimistic-concurrency" name="http-support-optimistic-concurrency">:white_check_mark:</a> **DO** support caching and optimistic concurrency by honoring the the `If-Match`, `If-None-Match`, if-modified-since, and if-unmodified-since request headers and by returning the ETag and last-modified response headers

#### HTTP Query Parameters and Header Values

<a href="#http-query-names-casing" name="http-query-names-casing">:white_check_mark:</a> **DO** use camel case for query parameter names.

Note: Certain legacy query parameter names use kebab-casing and are allowed only for backwards compatibility.

Because information in the service URL, as well as the request / response, are strings, there must be a predictable, well-defined scheme to convert strings to their corresponding values.

<a href="#http-parameter-validation" name="http-parameter-validation">:white_check_mark:</a> **DO** validate all query parameter and request header values and fail the operation with `400-Bad Request` if any value fails validation. Return an error response as described in the [Handling Errors](#handling-errors) section indicating what is wrong so customer can diagnose the issue and fix it themselves.

<a href="#http-parameter-serialization" name="http-parameter-serialization">:white_check_mark:</a> **DO** use the following table when translating strings:

Data type | Document that string must be
--------- | -------
Boolean   | true / false (all lowercase)
Integer   | -2<sup>53</sup>+1 to +2<sup>53</sup>-1 (for consistency with JSON limits on integers [RFC 8259](https://datatracker.ietf.org/doc/html/rfc8259))
Float     | [IEEE-754 binary64](https://en.wikipedia.org/wiki/Double-precision_floating-point_format)
String    | (Un)quoted?, max length, legal characters, case-sensitive, multiple delimiter
UUID      | 123e4567-e89b-12d3-a456-426614174000 (no {}s, hyphens, case-insensitive) [RFC 4122](https://datatracker.ietf.org/doc/html/rfc4122)
Date/Time (Header) | Sun, 06 Nov 1994 08:49:37 GMT [RFC 7231, Section 7.1.1.1](https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.1.1)
Date/Time (Query parameter) | YYYY-MM-DDTHH:mm:ss.sssZ (with at most 3 digits of fractional seconds) [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339)
Byte array | Base-64 encoded, max length
Array      | One of a) a comma-separated list of values (preferred), or b) separate `name=value` parameter instances for each value of the array


The table below lists the headers most used by Azure services:

Header Key          | Applies to | Example
------------------- | ---------- | -------------
_authorization_     | Request    | Bearer eyJ0...Xd6j (Support Azure Active Directory)
_x-ms-useragent_    | Request    | (see [Distributed Tracing & Telemetry](#distributed-tracing--telemetry))
traceparent         | Request    | (see [Distributed Tracing & Telemetry](#distributed-tracing--telemetry))
tracecontext        | Request    | (see [Distributed Tracing & Telemetry](#distributed-tracing--telemetry))
accept              | Request    | application/json
If-Match            | Request    | "67ab43" or * (no quotes) (see [Conditional Requests](#conditional-requests))
If-None-Match       | Request    | "67ab43" or * (no quotes) (see [Conditional Requests](#conditional-requests))
If-Modified-Since   | Request    | Sun, 06 Nov 1994 08:49:37 GMT (see [Conditional Requests](#conditional-requests))
If-Unmodified-Since | Request    | Sun, 06 Nov 1994 08:49:37 GMT (see [Conditional Requests](#conditional-requests))
date                | Both       | Sun, 06 Nov 1994 08:49:37 GMT (see [RFC 7231, Section 7.1.1.2](https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.1.2))
_content-type_      | Both       | application/merge-patch+json
_content-length_    | Both       | 1024
_x-ms-request-id_   | Response   | 4227cdc5-9f48-4e84-921a-10967cb785a0
ETag                | Response   | "67ab43" (see [Conditional Requests](#conditional-requests))
last-modified       | Response   | Sun, 06 Nov 1994 08:49:37 GMT
_x-ms-error-code_   | Response   | (see [Handling Errors](#handling-errors))
_azure-deprecating_ | Response   | (see [Deprecating Behavior Notification](#deprecating-behavior-notification))
retry-after         | Response   | 180 (see [RFC 7231, Section 7.1.3](https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.3))

<a href="#http-header-support-standard-headers" name="http-header-support-standard-headers">:white_check_mark:</a> **DO** support all headers shown in _italics_

<a href="#http-header-names-casing" name="http-header-names-casing">:white_check_mark:</a> **DO** specify headers using kebab-casing

<a href="#http-header-names-case-sensitivity" name="http-header-names-case-sensitivity">:white_check_mark:</a> **DO** compare request header names using case-insensitivity

<a href="#http-header-values-case-sensitivity" name="http-header-values-case-sensitivity">:white_check_mark:</a> **DO** compare request header values using case-sensitivity if the header name requires it

<a href="#http-header-date-values" name="http-header-date-values">:white_check_mark:</a> **DO** accept date values in headers in HTTP-Date format and return date values in headers in the IMF-fixdate format as defined in [RFC 7231, Section 7.1.1.1](https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.1.1), e.g. "Sun, 06 Nov 1994 08:49:37 GMT".

Note: The RFC 7231 IMF-fixdate format is a "fixed-length and single-zone subset" of the RFC 1123 / RFC 5822 format, which means: a) year must be four digits, b) the seconds component of time is required, and c) the timezone must be GMT.

<a href="#http-header-request-id" name="http-header-request-id">:white_check_mark:</a> **DO** create an opaque value that uniquely identifies the request and return this value in the `x-ms-request-id` response header.

Your service should include the `x-ms-request-id` value in error logs so that users can submit support requests for specific failures using this value.

<a href="#http-allow-unrecognized-headers" name="http-allow-unrecognized-headers">:no_entry:</a> **DO NOT** fail a request that contains an unrecognized header. Headers may be added by API gateways or middleware and this must be tolerated

<a href="#http-no-x-custom-headers" name="http-no-x-custom-headers">:no_entry:</a> **DO NOT** use "x-" prefix for custom headers, unless the header already exists in production [[RFC 6648](https://datatracker.ietf.org/doc/html/rfc6648)].

**Additional References**
- [StackOverflow - Difference between http parameters and http headers](https://stackoverflow.com/questions/40492782)
- [Standard HTTP Headers](https://httpwg.org/specs/rfc7231.html#header.field.registration)
- [Why isn't HTTP PUT allowed to do partial updates in a REST API?](https://stackoverflow.com/questions/19732423/why-isnt-http-put-allowed-to-do-partial-updates-in-a-rest-api)

<a href="#rest" name="rest"></a>
### REpresentational State Transfer (REST)
REST is an architectural style with broad reach that emphasizes scalability, generality, independent deployment, reduced latency via caching, and security. When applying REST to your API, you define your service’s resources as a collections of items.
These are typically the nouns you use in the vocabulary of your service. Your service's [URLs](#uniform-resource-locators-urls) determine the hierarchical path developers use to perform CRUD (create, read, update, and delete) operations on resources. Note, it's important to model resource state, not behavior.
There are patterns, later in these guidelines, that describe how to invoke behavior on your service. See [this article in the Azure Architecture Center](https://docs.microsoft.com/azure/architecture/best-practices/api-design) for a more detailed discussion of REST API design patterns.

When designing your service, it is important to optimize for the developer using your API.

<a href="#rest-clear-naming" name="rest-clear-naming">:white_check_mark:</a> **DO** focus heavily on clear & consistent naming

<a href="#rest-paths-make-sense" name="rest-paths-make-sense">:white_check_mark:</a> **DO** ensure your resource paths make sense

<a href="#rest-simplify-operations" name="rest-simplify-operations">:white_check_mark:</a> **DO** simplify operations with few required query parameters & JSON fields

<a href="#rest-specify-string-value-constraints" name="rest-specify-string-value-constraints">:white_check_mark:</a> **DO** establish clear contracts for string values

<a href="#rest-use-standard-status-codes" name="rest-use-standard-status-codes">:white_check_mark:</a> **DO** use proper response codes/bodies so customer can diagnose their own problems and fix them without contacting Azure support or the service team

#### Resource Schema & Field Mutability

<a href="#rest-response-body-is-resource-schema" name="rest-response-body-is-resource-schema">:white_check_mark:</a> **DO** use the same JSON schema for PUT request/response, PATCH response, GET response, and POST request/response on a given URL path. The PATCH request schema should contain all the same fields with no required fields. This allows one SDK type for input/output operations and enables the response to be passed back in a request.

<a href="#rest-field-mutability" name="rest-field-mutability">:white_check_mark:</a> **DO** think about your resource's fields and how they are used:

Field Mutability | Service Request's behavior for this field
-----------------| -----------------------------------------
**Create** | Service honors field only when creating a resource. Minimize create-only fields so customers don't have to delete & re-create the resource.
**Update** | Service honors field when creating or updating a resource
**Read**   | Service returns this field in a response. If the client passed a read-only field, the service **MUST** fail the request unless the passed-in value matches the resource's current value

In addition to the above, a field may be "required" or "optional". A required field is guaranteed to always exist and will typically _not_ become a nullable field in a SDK's data structure. This allows customers to write code without performing a null-check.
Because of this, required fields can only be introduced in the 1st version of a service; it is a breaking change to introduce required fields in a later version. In addition, it is a breaking change to remove a required field or make an optional field required or vice versa.

<a href="#rest-flat-is-better-than-nested" name="rest-flat-is-better-than-nested">:white_check_mark:</a> **DO** make fields simple and maintain a shallow hierarchy.

<a href="#rest-get-returns-json-body" name="rest-get-returns-json-body">:white_check_mark:</a> **DO** use GET for resource retrieval and return JSON in the response body

<a href="#rest-patch-use-merge-patch" name="rest-patch-use-merge-patch">:white_check_mark:</a> **DO** create and update resources using PATCH [RFC 5789] with JSON Merge Patch [(RFC 7396)](https://datatracker.ietf.org/doc/html/rfc7396) request body.

<a href="#rest-put-for-create-or-replace" name="rest-put-for-create-or-replace">:white_check_mark:</a> **DO** use PUT with JSON for wholesale create/replace operations. **NOTE:** If a v1 client PUTs a resource; any fields introduced in V2+ should be reset to their default values (the equivalent to DELETE followed by PUT).

<a href="#rest-delete-resource" name="rest-delete-resource">:white_check_mark:</a> **DO** use DELETE to remove a resource.

<a href="#rest-fail-for-unknown-fields" name="rest-fail-for-unknown-fields">:white_check_mark:</a> **DO** fail an operation with `400-Bad Request` if the request is improperly-formed or if any JSON field name or value is not fully understood by the specific version of the service. Return an error response as described in [Handling errors](#handling-errors) indicating what is wrong so customer can diagnose the issue and fix it themselves.

<a href="#rest-secrets-allowed-in-post-response" name="rest-secrets-allowed-in-post-response">:heavy_check_mark:</a> **YOU MAY** return secret fields via POST **if absolutely necessary**.

<a href="#rest-no-secrets-in-get-response" name="rest-no-secrets-in-get-response">:no_entry:</a> **DO NOT** return secret fields via GET. For example, do not return `administratorPassword` in JSON.

<a href="#rest-no-computable-fields" name="rest-no-computable-fields">:no_entry:</a> **DO NOT** add fields to the JSON if the value is easily computable from other fields to avoid bloating the body.

##### Create / Update / Replace Processing Rules

<a href="#rest-put-patch-status-codes" name="rest-put-patch-status-codes">:white_check_mark:</a> **DO** follow the processing below to create/update/replace a resource:

When using this method | if this condition happens | use&nbsp;this&nbsp;response&nbsp;code
---------------------- | ------------------------- | ----------------------
PATCH/PUT | Any JSON field name/value not known/valid to the api-version | `400-Bad Request`
PATCH/PUT | Any Read field passed (client can't set Read fields) | `400-Bad Request`
| **If&nbsp;the&nbsp;resource&nbsp;does&nbsp;not&nbsp;exist** |
PATCH/PUT | Any mandatory Create/Update field missing | `400-Bad Request`
PATCH/PUT | Create resource using Create/Update fields | `201-Created`
| **If&nbsp;the&nbsp;resource&nbsp;already&nbsp;exists** |
PATCH | Any Create field doesn't match current value (allows retries) | `409-Conflict`
PATCH | Update resource using Update fields | `200-OK`
PUT | Any mandatory Create/Update field missing | `400-Bad Request`
PUT | Overwrite resource entirely using Create/Update fields | `200-OK`

#### Handling Errors
There are 2 kinds of errors:
- An error where you expect customer code to gracefully recover at runtime
- An error indicating a bug in customer code that is unlikely to be recoverable at runtime; the customer must just fix their code

<a href="#rest-error-code-header" name="rest-error-code-header">:white_check_mark:</a> **DO** return an `x-ms-error-code` response header with a string error code indicating what went wrong.

*NOTE: `x-ms-error-code` values are part of your API contract (because customer code is likely to do comparisons against them) and cannot change in the future.*

<a href="#rest-error-code-enum" name="rest-error-code-enum">:heavy_check_mark:</a> **YOU MAY** implement the `x-ms-error-code` values as an enum with `"modelAsString": true` because it's possible add new values over time.  In particular, it's only a breaking change if the same conditions result in a *different* top-level error code.

<a href="#rest-add-codes-in-new-api-version" name="rest-add-codes-in-new-api-version">:warning:</a> **YOU SHOULD NOT** add new top-level error codes to an existing API without bumping the service version.

<a href="#rest-descriptive-error-code-values" name="rest-descriptive-error-code-values">:white_check_mark:</a> **DO** carefully craft unique `x-ms-error-code` string values for errors that are recoverable at runtime.  Reuse common error codes for usage errors that are not recoverable.

<a href="#rest-error-code-grouping" name="rest-error-code-grouping">:heavy_check_mark:</a> **YOU MAY** group common customer code errors into a few `x-ms-error-code` string values.

<a href="#rest-error-code-header-and-body-match" name="rest-error-code-header-and-body-match">:white_check_mark:</a> **DO** ensure that the top-level error's `code` value is identical to the `x-ms-error-code` header's value.

<a href="#rest-error-response-body-structure" name="rest-error-response-body-structure">:white_check_mark:</a> **DO** provide a response body with the following structure:

**ErrorResponse** : Object

Property | Type | Required | Description
-------- | ---- | :------: | -----------
`error` | ErrorDetail | ✔ | The top-level error object whose `code` matches the `x-ms-error-code` response header

**ErrorDetail** : Object

Property | Type | Required | Description
-------- | ---- | :------: | -----------
`code` | String | ✔ | One of a server-defined set of error codes.
`message` | String | ✔ | A human-readable representation of the error.
`target` | String |  | The target of the error.
`details` | ErrorDetail[] |  | An array of details about specific errors that led to this reported error.
`innererror` | InnerError |  | An object containing more specific information than the current object about the error.
_additional properties_ |   | | Additional properties that can be useful when debugging.

**InnerError** : Object

Property | Type | Required | Description
-------- | ---- | :------: | -----------
`code` | String |  | A more specific error code than was provided by the containing error.
`innererror` | InnerError |  | An object containing more specific information than the current object about the error.

Example:
```json
{
  "error": {
    "code": "InvalidPasswordFormat",
    "message": "Human-readable description",
    "target": "target of error",
    "innererror": {
      "code": "PasswordTooShort",
      "minLength": 6,
    }
  }
}
```

<a href="#rest-document-error-code-values" name="rest-document-error-code-values">:white_check_mark:</a> **DO** document the service's top-level error code strings; they are part of the API contract.

<a href="#rest-error-non-api-contract-fields" name="rest-error-non-api-contract-fields">:heavy_check_mark:</a> **YOU MAY** treat the other fields as you wish as they are _not_ considered part of your service's API contract and customers should not take a dependency on them or their value. They exist to help customers self-diagnose issues.

<a href="#rest-error-additional-properties-allowed" name="rest-error-additional-properties-allowed">:heavy_check_mark:</a> **YOU MAY** add additional properties for any data values in your error message so customers don't resort to parsing your error message.  For example, an error with `"message": "A maximum of 16 keys are allowed per account."` might also add a `"maximumKeys": 16` property.  This is not part of your API contract and should only be used for diagnosing problems.

*Note: Do not use this mechanism to provide information developers need to rely on in code (ex: the error message can give details about why you've been throttled, but the `Retry-After` should be what developers rely on to back off).*

<a href="#rest-error-use-default-response" name="rest-error-use-default-response">:warning:</a> **YOU SHOULD NOT** document specific error status codes in your OpenAPI/Swagger spec unless the "default" response cannot properly describe the specific error response (e.g. body schema is different).

<a href="#json" name="json"></a>
### JSON

<a href="#json-field-name-casing" name="json-field-name-casing">:white_check_mark:</a> **DO** use camel case for all JSON field names. Do not upper-case acronyms; use camel case.

<a href="#json-field-names-case-sensitivity" name="json-field-names-case-sensitivity">:white_check_mark:</a> **DO** treat JSON field names with case-sensitivity.

<a href="#json-field-values-case-sensitivity" name="json-field-values-case-sensitivity">:white_check_mark:</a> **DO** treat JSON field values with case-sensitivity. There may be some exceptions but avoid if at all possible.

<a href="#json-field-values-id" name="json-field-values-ids">:white_check_mark:</a> **DO** treat JSON field value representing a unique ID as an opaque string value and compare them with case-sensitivity. For example, IDs are frequently [UUIDs](https://en.wikipedia.org/wiki/Universally_unique_identifier), [CUIDs](https://github.com/paralleldrive/cuid2), [Nano ID](https://blog.openapihub.com/en-us/what-is-nano-id-its-difference-from-uuid-as-unique-identifiers/), or other formats. The choice of ID format is a service implementation detail. Customer code should only ever need to get, store, and send these values and should never perform any other kind of parsing or interpretation of ID values.

Services, and the clients that access them, may be written in multiple languages. To ensure interoperability, JSON establishes the "lowest common denominator" type system, which is always sent over the wire as UTF-8 bytes. This system is very simple and consists of three types:

 Type | Description
 ---- | -----------
 Boolean | true/false (always lowercase)
 Number  | Signed floating point (IEEE-754 binary64; int range: -2<sup>53</sup>+1 to +2<sup>53</sup>-1)
 String  | Used for everything else

<a href="#json-null-response-values" name="json-null-response-values">:no_entry:</a> **DO NOT** send JSON fields with a null value from the service to the client. Instead, the service should just not send this field at all (this reduces payload size). Semantically, Azure services treat a missing field and a field with a value of null as identical. 

<a href="#json-null-request-values" name="json-null-resquest-values">:white_check_mark:</a> **DO** accept JSON fields with a null value only for a PATCH operation with a JSON Merge Patch payload. A field with a value of null instructs the service to delete the field. If the field cannot be deleted, then return 400-BadRequest, else return the resource with the deleted field missing from the response payload (see bullet above).

<a href="#json-integer-values" name="json-integer-values">:white_check_mark:</a> **DO** use integers within the acceptable range of JSON number.

<a href="#json-specify-string-constraints" name="json-specify-string-constraints">:white_check_mark:</a> **DO** establish a well-defined contract for the format of strings. For example, determine minimum length, maximum length, legal characters, case-(in)sensitive comparisons, etc. Where possible, use standard formats, e.g. RFC 3339 for date/time.

<a href="#json-use-standard-string-formats" name="json-use-standard-string-formats">:white_check_mark:</a> **DO** use strings formats that are well-known and easily parsable/formattable by many programming languages, e.g. RFC 3339 for date/time.

<a href="#json-should-be-round-trippable" name="json-should-be-round-trippable">:white_check_mark:</a> **DO** ensure that information exchanged between your service and any client is "round-trippable" across multiple programming languages.

<a href="#json-date-time-is-rfc3339" name="json-date-time-is-rfc3339">:white_check_mark:</a> **DO** use [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) for date/time.

<a href="#json-durations-use-fixed-time-intervals" name="json-durations-use-fixed-time-intervals">:white_check_mark:</a> **DO** use a fixed time interval to express durations e.g., milliseconds, seconds, minutes, days, etc., and include the time unit in the property name e.g., `backupTimeInMinutes` or `ttlSeconds`.

<a href="#json-rfc3339-time-intervals-allowed" name="json-rfc3339-time-intervals-allowed">:heavy_check_mark:</a> **YOU MAY** use [RFC 3339 time intervals](https://wikipedia.org/wiki/ISO_8601#Durations) only when users must be able to specify a time interval that may change from month to month or year to year e.g., "P3M" represents 3 months no matter how many days between the start and end dates, or "P1Y" represents 366 days on a leap year. The value must be round-trippable.

<a href="#json-uuid-is-rfc4412" name="json-uuid-is-rfc4412">:white_check_mark:</a> **DO** use [RFC 4122](https://datatracker.ietf.org/doc/html/rfc4122) for UUIDs.

<a href="#json-may-nest-for-grouping" name="json-may-nest-for-grouping">:heavy_check_mark:</a> **YOU MAY** use JSON objects to group sub-fields together.

<a href="#json-use-arrays-for-ordering" name="json-use-arrays-for-ordering">:heavy_check_mark:</a> **YOU MAY** use JSON arrays if maintaining an order of values is required. Avoid arrays in other situations since arrays can be difficult and inefficient to work with, especially with JSON Merge Patch where the entire array needs to be read prior to any operation being applied to it.

<a href="#json-prefer-objects-over-arrays" name="json-prefer-objects-over-arrays">:ballot_box_with_check:</a> **YOU SHOULD** use JSON objects instead of arrays whenever possible.

#### Enums & SDKs (Client libraries)
It is common for strings to have an explicit set of values. These are often reflected in the OpenAPI definition as enumerations. These are extremely useful for developer tooling, e.g. code completion, and client library generation.

However, it is not uncommon for the set of values to grow over the life of a service. For this reason, Microsoft's tooling uses the concept of an "extensible enum," which indicates that the set of values should be treated as only a _partial_ list.
This indicates to client libraries and customers that values of the enumeration field should be effectively treated as strings and that undocumented value may returned in the future. This enables the set of values to grow over time while ensuring stability in client libraries and customer code.

<a href="#json-use-extensible-enums" name="json-use-extensible-enums">:ballot_box_with_check:</a> **YOU SHOULD** use extensible enumerations unless you are positive that the symbol set will NEVER change over time.

<a href="#json-document-extensible-enums" name="json-document-extensible-enums">:white_check_mark:</a> **DO** document to customers that new values may appear in the future so that customers write their code today expecting these new values tomorrow.

<a href="#json-return-extensible-enum-value" name="json-return-extensible-enum-value">:heavy_check_mark:</a> **YOU MAY** return a value for an extensible enum that is not one of the values defined for the api-version specified in the request.

<a href="#json-accept-extensible-enum-value" name="json-accept-extensible-enum-value">:warning:</a> **YOU SHOULD NOT** accept a value for an extensible enum that is not one of the values defined for the api-version specified in the request.

<a href="#json-removing-enum-value-is-breaking" name="json-removing-enum-value-is-breaking">:no_entry:</a> **DO NOT** remove values from your enumeration list as this breaks customer code.

#### Polymorphic types

Polymorphism types in REST APIs refers to the possibility to use the same property of a request or response to have similar but different shapes. This is commonly expressed as a `oneOf` in JsonSchema or OpenAPI. In order to simplify how to determine which specific type a given request or response payload corresponds to, Azure requires the use of an explicit discriminator field.

Note: Polymorphic types can make your service more difficult for nominally typed languages to consume. See the corresponding section in the [Considerations for service design](./ConsiderationsForServiceDesign.md#avoid-surprises) for more information.

<a href="#json-use-discriminator-for-polymorphism" name="json-use-discriminator-for-polymorphism">:white_check_mark:</a> **DO** define a discriminator field indicating the kind of the resource and include any kind-specific fields in the body.

Below is an example of JSON for a Rectangle and Circle with a discriminator field named `kind`:

**Rectangle**
```json
{
   "kind": "rectangle",
   "x": 100,
   "y": 50,
   "width": 10,
   "length": 24,
   "fillColor": "Red",
   "lineColor": "White",
   "subscription": {
      "kind": "free"
   }
}
```

**Circle**
```json
{
   "kind": "circle",
   "x": 100,
   "y": 50,
   "radius": 10,
   "fillColor": "Green",
   "lineColor": "Black",
   "subscription": {
      "kind": "paid",
      "expiration": "2024",
      "invoice": "123456"
   }
}
```
Both Rectangle and Circle have common fields: `kind`, `fillColor`, `lineColor`, and `subscription`. A Rectangle also has `x`, `y`, `width`, and `length` while a Circle has `x`, `y`, and `radius`. The `subscription` is a nested polymorphic type. A `free` subscription has no additional fields and a `paid` subscription has `expiration` and `invoice` fields.

The [Azure Naming Guidelines](./ConsiderationsForServiceDesign.md#common-names) recommend that the discriminator field be named `kind`.

<a href="#json-polymorphism-kind-extensible" name="json-polymorphism-kind-extensible">:ballot_box_with_check:</a> **YOU SHOULD** define the discriminator field of a polymorphic type to be an extensible enum.

<a href="#json-polymorphism-kind-immutable" name="json-polymorphism-kind-immutable">:warning:</a> **YOU SHOULD NOT** allow an update (patch) to change the discriminator field of a polymorphic type.

<a href="#json-polymorphism-versioning" name="json-polymorphism-versioning">:warning:</a> **YOU SHOULD NOT** return properties of a polymorphic type that are not defined for the api-version specified in the request.

<a href="#json-polymorphism-arrays" name="json-polymorphism-arrays">:warning:</a> **YOU SHOULD NOT** have a property of an updatable resource whose value is an array of polymorphic objects.

Updating an array property with JSON merge-patch is not version-resilient if the array contains polymorphic types.

## Common API Patterns

<a href="#actions" name="actions"></a>
### Performing an Action
The REST specification is used to model the state of a resource, and is primarily intended to handle CRUD (Create, Read, Update, Delete) operations. However, many services require the ability to perform an action on a resource, e.g. getting the thumbnail of an image or rebooting a VM.  It is also sometimes useful to perform an action on a collection.

<a href="#actions-url-pattern-for-resource-action" name="actions-url-pattern-for-resource-action">:ballot_box_with_check:</a> **YOU SHOULD** pattern your URL like this to perform an action on a resource
**URL Pattern**
```text
https://.../<resource-collection>/<resource-id>:<action>?<input parameters>
```

**Example**
```text
https://.../users/Bob:grant?access=read
```

<a href="#actions-url-pattern-for-collection-action" name="actions-url-pattern-for-collection-action">:ballot_box_with_check:</a> **YOU SHOULD** pattern your URL like this to perform an action on a collection
**URL Pattern**
```text
https://.../<resource-collection>:<action>?<input parameters>
```

**Example**
```text
https://.../users:grant?access=read
```

Note: To avoid potential collision of actions and resource ids, you should disallow the use of the ":" character in resource ids.

<a href="#actions-use-post-method" name="actions-use-post-method">:white_check_mark:</a> **DO** use a POST operation for any action on a resource or collection.

<a href="#actions-support-repeatability-headers" name="actions-support-repeatability-headers">:white_check_mark:</a> **DO** support the Repeatability-Request-ID & Repeatability-First-Sent request headers if the action needs to be idempotent if retries occur.

<a href="#actions-synchronous-success-status-code" name="actions-synchronous-success-status-code">:white_check_mark:</a> **DO** return a `200-OK` when the action completes synchronously and successfully.

<a href="#actions-action-name-is-verb" name="actions-action-name-is-verb">:ballot_box_with_check:</a> **YOU SHOULD** use a verb as the `<action>` component of the path.

<a href="#actions-no-actions-for-crud" name="actions-no-actions-for-crud">:no_entry:</a> **DO NOT** use an action operation when the operation behavior could reasonably be defined as one of the standard REST Create, Read, Update, Delete, or List operations.

<a href="#collections" name="collections"></a>
### Collections
<a href="#collections-response-is-object" name="collections-response-is-object">:white_check_mark:</a> **DO** structure the response to a list operation as an object with a top-level array field containing the set (or subset) of resources.

<a href="#collections-support-server-driven-paging" name="collections-support-server-driven-paging">:ballot_box_with_check:</a> **YOU SHOULD** support paging today if there is ever a chance in the future that the number of items can grow to be very large.

NOTE: It is a breaking change to add paging in the future

<a href="#collections-use-get-method" name="collections-use-get-method">:heavy_check_mark:</a> **YOU MAY** expose an operation that lists your resources by supporting a GET method with a URL to a resource-collection (as opposed to a resource-id).

**Example Response Body**
```json
{
    "value": [
       { "id": "Item 01", "etag": "\"abc\"", "price": 99.95, "size": "Medium" },
       { … },
       { … },
       { "id": "Item 99", "etag": "\"def\"", "price": 59.99, "size": "Large" }
    ],
    "nextLink": "{opaqueUrl}"
 }
```

<a href="#collections-items-have-id-and-etag" name="collections-items-have-id-and-etag">:white_check_mark:</a> **DO** include the _id_ field and _etag_ field (if supported) for each item as this allows the customer to modify the item in a future operation. Note that the etag field _must_ have escaped quotes embedded within it; for example, "\"abc\"" or W/"\"abc\"".

<a href="#collections-document-pagination-reliability" name="collections-document-pagination-reliability">:white_check_mark:</a> **DO** clearly document that resources may be skipped or duplicated across pages of a paginated collection unless the operation has made special provisions to prevent this (like taking a time-expiring snapshot of the collection).

<a href="#collections-include-nextlink-for-more-results" name="collections-include-nextlink-for-more-results">:white_check_mark:</a> **DO** return a `nextLink` field with an absolute URL that the client can GET in order to retrieve the next page of the collection.

Note: The service is responsible for performing any URL-encoding required on the `nextLink` URL.

<a href="#collections-nextlink-includes-all-query-params" name="collections-nextlink-includes-all-query-params">:white_check_mark:</a> **DO** include any query parameters required by the service in `nextLink`, including `api-version`.

<a href="#collections-response-array-name" name="collections-response-array-name">:ballot_box_with_check:</a> **YOU SHOULD** use `value` as the name of the top-level array field unless a more appropriate name is available.

<a href="#collections-no-nextlink-on-last-page" name="collections-no-nextlink-on-last-page">:no_entry:</a> **DO NOT** return the `nextLink` field at all when returning the last page of the collection.

<a href="#collections-nextlink-value-never-null" name="collections-nextlink-value-never-null">:no_entry:</a> **DO NOT** return the `nextLink` field with a value of null.

<a href="#collections-avoid-count-property" name="collections-avoid-count-property">:warning:</a> **YOU SHOULD NOT** return a `count` of all objects in the collection as this may be expensive to compute.

#### Query options

<a href="#collections-query-options" name="collections-query-options">:heavy_check_mark:</a> **YOU MAY** support the following query parameters allowing customers to control the list operation:

Parameter&nbsp;name | Type | Description
------------------- | ---- | -----------
`filter`       | string            | an expression on the resource type that selects the resources to be returned
`orderby`      | string&nbsp;array | a list of expressions that specify the order of the returned resources
`skip`         | integer           | an offset into the collection of the first resource to be returned
`top`          | integer           | the maximum number of resources to return from the collection
`maxpagesize`  | integer           | the maximum number of resources to include in a single response
`select`       | string&nbsp;array | a list of field names to be returned for each resource
`expand`       | string&nbsp;array | a list of the related resources to be included in line with each resource

<a href="#collections-error-on-unknown-parameter" name="collections-error-on-unknown-parameter">:white_check_mark:</a> **DO** return an error if the client specifies any parameter not supported by the service.

<a href="#collections-parameter-names-case-sensitivity" name="collections-parameter-names-case-sensitivity">:white_check_mark:</a> **DO** treat these query parameter names as case-sensitive.

<a href="#collections-select-expand-ordering" name="collections-select-expand-ordering">:white_check_mark:</a> **DO** apply `select` or `expand` options after applying all the query options in the table above.

<a href="#collections-query-options-ordering" name="collections-query-options-ordering">:white_check_mark:</a> **DO** apply the query options to the collection in the order shown in the table above.

<a href="#collections-query-options-no-dollar-sign" name="collections-query-options-no-dollar-sign">:no_entry:</a> **DO NOT** prefix any of these query parameter names with "$" (the convention in the [OData standard](http://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html#sec_QueryingCollections)).

#### filter

<a href="#collections-filter-param" name="collections-filter-param">:heavy_check_mark:</a> **YOU MAY** support filtering of the results of a list operation with the `filter` query parameter.

The value of the `filter` query parameter is an expression involving the fields of the resource that produces a Boolean value. This expression is evaluated for each resource in the collection and only items where the expression evaluates to true are included in the response.

<a href="#collections-filter-behavior" name="collections-filter-behavior">:white_check_mark:</a> **DO** omit all resources from the collection for which the filter expression evaluates to false or to null, or references properties that are unavailable due to permissions.

Example: return all Products whose Price is less than $10.00

```text
GET https://api.contoso.com/products?filter=price lt 10.00
```

##### filter operators

:heavy_check_mark: **YOU MAY** support the following operators in filter expressions:

Operator                 | Description           | Example
--------------------     | --------------------- | -----------------------------------------------------
**Comparison Operators** |                       |
eq                       | Equal                 | city eq 'Redmond'
ne                       | Not equal             | city ne 'London'
gt                       | Greater than          | price gt 20
ge                       | Greater than or equal | price ge 10
lt                       | Less than             | price lt 20
le                       | Less than or equal    | price le 100
**Logical Operators**    |                       |
and                      | Logical and           | price le 200 and price gt 3.5
or                       | Logical or            | price le 3.5 or price gt 200
not                      | Logical negation      | not price le 3.5
**Grouping Operators**   |                       |
( )                      | Precedence grouping   | (priority eq 1 or city eq 'Redmond') and price gt 100

<a href="#collections-filter-unknown-operator" name="collections-filter-unknown-operator">:white_check_mark:</a> **DO** respond with an error message as defined in the [Handling Errors](#handling-errors) section if a client includes an operator in a filter expression that is not supported by the operation.

<a href="#collections-filter-operator-ordering" name="collections-filter-operator-ordering">:white_check_mark:</a> **DO** use the following operator precedence for supported operators when evaluating filter expressions. Operators are listed by category in order of precedence from highest to lowest. Operators in the same category have equal precedence and should be evaluated left to right:

| Group           | Operator | Description
| ----------------|----------|------------
| Grouping        | ( )      | Precedence grouping   |
| Unary           | not      | Logical Negation      |
| Relational      | gt       | Greater Than          |
|                 | ge       | Greater than or Equal |
|                 | lt       | Less Than             |
|                 | le       | Less than or Equal    |
| Equality        | eq       | Equal                 |
|                 | ne       | Not Equal             |
| Conditional AND | and      | Logical And           |
| Conditional OR  | or       | Logical Or            |

<a href="#collections-filter-functions" name="collections-filter-functions">:heavy_check_mark:</a> **YOU MAY** support orderby and filter functions such as concat and contains. For more information, see [odata Canonical Functions](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part2-url-conventions.html#_Toc31360979).

##### Operator examples
The following examples illustrate the use and semantics of each of the logical operators.

Example: all products with a name equal to 'Milk'

```text
GET https://api.contoso.com/products?filter=name eq 'Milk'
```

Example: all products with a name not equal to 'Milk'

```text
GET https://api.contoso.com/products?filter=name ne 'Milk'
```

Example: all products with the name 'Milk' that also have a price less than 2.55:

```text
GET https://api.contoso.com/products?filter=name eq 'Milk' and price lt 2.55
```

Example: all products that either have the name 'Milk' or have a price less than 2.55:

```text
GET https://api.contoso.com/products?filter=name eq 'Milk' or price lt 2.55
```

Example: all products that have the name 'Milk' or 'Eggs' and have a price less than 2.55:

```text
GET https://api.contoso.com/products?filter=(name eq 'Milk' or name eq 'Eggs') and price lt 2.55
```

#### orderby

<a href="#collections-orderby-param" name="collections-orderby-param">:heavy_check_mark:</a> **YOU MAY** support sorting of the results of a list operation with the `orderby` query parameter.
*NOTE: It is unusual for a service to support `orderby` because it is very expensive to implement as it requires sorting the entire large collection before being able to return any results.*

The value of the `orderby` parameter is a comma-separated list of expressions used to sort the items.
A special case of such an expression is a property path terminating on a primitive property.

Each expression in the `orderby` parameter value may include the suffix "asc" for ascending or "desc" for descending, separated from the expression by one or more spaces.

<a href="#collections-orderby-ordering" name="collections-orderby-ordering">:white_check_mark:</a> **DO** sort the collection in ascending order on an expression if "asc" or "desc" is not specified.

<a href="#collections-orderby-null-ordering" name="collections-orderby-null-ordering">:white_check_mark:</a> **DO** sort NULL values as "less than" non-NULL values.

<a href="#collections-orderby-behavior" name="collections-orderby-behavior">:white_check_mark:</a> **DO** sort items by the result values of the first expression, and then sort items with the same value for the first expression by the result value of the second expression, and so on.

<a href="#collections-orderby-inherent-sort-order" name="collections-orderby-inherent-sort-order">:white_check_mark:</a> **DO** use the inherent sort order for the type of the field. For example, date-time values should be sorted chronologically and not alphabetically.

<a href="#collections-orderby-unsupported-field" name="collections-orderby-unsupported-field">:white_check_mark:</a> **DO** respond with an error message as defined in the [Handling Errors](#handling-errors) section if the client requests sorting by a field that is not supported by the operation.

For example, to return all people sorted by name in ascending order:
```text
GET https://api.contoso.com/people?orderby=name
```

For example, to return all people sorted by name in descending order and a secondary sort order of hireDate in ascending order.
```text
GET https://api.contoso.com/people?orderby=name desc,hireDate
```

Sorting MUST compose with filtering such that:
```text
GET https://api.contoso.com/people?filter=name eq 'david'&orderby=hireDate
```
will return all people whose name is David sorted in ascending order by hireDate.

##### Considerations for sorting with pagination

<a href="#collections-consistent-options-with-pagination" name="collections-consistent-options-with-pagination">:white_check_mark:</a> **DO** use the same filtering options and sort order for all pages of a paginated list operation response.

##### skip
<a href="#collections-skip-param-definition" name="collections-skip-param-definition">:white_check_mark:</a> **DO** define the `skip` parameter as an integer with a default and minimum value of 0.

<a href="#collections-skip-param" name="collections-skip-param">:heavy_check_mark:</a> **YOU MAY** allow clients to pass the `skip` query parameter to specify an offset into collection of the first resource to be returned.
##### top

<a href="#collections-" name="collections-"></a>
<a href="#collections-top-param" name="collections-top-param">:heavy_check_mark:</a> **YOU MAY** allow clients to pass the `top` query parameter to specify the maximum number of resources to return from the collection.

If supporting `top`:
:white_check_mark: **DO** define the `top` parameter as an integer with a minimum value of 1. If not specified, `top` has a default value of infinity.

<a href="#collections-top-behavior" name="collections-top-behavior">:white_check_mark:</a> **DO** return the collection's `top` number of resources (if available), starting from `skip`.

##### maxpagesize

<a href="#collections-maxpagesize-param" name="collections-maxpagesize-param">:heavy_check_mark:</a> **YOU MAY** allow clients to pass the `maxpagesize` query parameter to specify the maximum number of resources to include in a single page response.

<a href="#collections-maxpagesize-definition" name="collections-maxpagesize-definition">:white_check_mark:</a> **DO** define the `maxpagesize` parameter as an optional integer with a default value appropriate for the collection.

<a href="#collections-maxpagesize-might-return-fewer" name="collections-maxpagesize-might-return-fewer">:white_check_mark:</a> **DO** make clear in documentation of the `maxpagesize` parameter that the operation may choose to return fewer resources than the value specified.

<a href="#versioning" name="versioning"></a>
### API Versioning

Azure services need to change over time. However, when changing a service, there are 2 requirements:
 1. Already-running customer workloads must not break due to a service change
 2. Customers can adopt a new service version without requiring any code changes (Of course, the customer must modify code to leverage any new service features.)

*NOTE: the [Azure Breaking Change Policy](http://aka.ms/AzBreakingChangesPolicy) has tables (section 5) describing what kinds of changes are considered breaking. Breaking changes are allowable (due to security/compliance/etc.) if approved by the [Azure Breaking Change Reviewers](mailto:azbreakchangereview@microsoft.com) but only following ample communication to customers and a lengthy deprecation period.*

<a href="#versioning-review-required" name="versioning-review-required">:white_check_mark:</a> **DO** review any API changes with the Azure API Stewardship Board

Clients specify the version of the API to be used in every request to the service, even requests to an `Operation-Location` or `nextLink` URL returned by the service.

<a href="#versioning-api-version-query-param" name="versioning-api-version-query-param">:white_check_mark:</a> **DO** use a required query parameter named `api-version` on every operation for the client to specify the API version.

<a href="#versioning-date-based-versioning" name="versioning-date-based-versioning">:white_check_mark:</a> **DO** use `YYYY-MM-DD` date values, with a `-preview` suffix for preview versions, as the valid values for `api-version`.

<a href="#versioning-api-version-missing" name="versioning-api-version-missing">:white_check_mark:</a> **DO** return HTTP 400 with error code "MissingApiVersionParameter" and message "The api-version query parameter (?api-version=) is required for all requests" if client omits the `api-version` query parameter.

<a href="#versioning-api-version-unsupported" name="versioning-api-version-unsupported">:white_check_mark:</a> **DO** return HTTP 400 with error code "UnsupportedApiVersionValue" and message "Unsupported api-version '{0}'. The supported api-versions are '{1}'." if client passes an `api-version` value unrecognized by the service. For the supported api-versions, just list all the stable versions still supported by the service and just the latest public preview version (if any).

```text
PUT https://service.azure.com/users/Jeff?api-version=2021-06-04
```

<a href="#versioning-use-later-date" name="versioning-use-later-date">:white_check_mark:</a> **DO** use a later date for each new preview version

When releasing a new preview, the service team may completely retire any previous preview versions after giving customers at least 90 days to upgrade their code

<a href="#versioning-no-breaking-changes" name="versioning-no-breaking-changes">:no_entry:</a> **DO NOT** introduce any breaking changes into the service.

<a href="#versioning-no-version-in-path" name="versioning-no-version-in-path">:no_entry:</a> **DO NOT** include a version number segment in any operation path.

<a href="#versioning-use-later-date-2" name="versioning-use-later-date-2">:no_entry:</a> **DO NOT** use the same date when transitioning from a preview API to a GA API. If the preview `api-version` is '2021-06-04-preview', the GA version of the API **must be** a date later than 2021-06-04

<a href="#versioning-preview-goes-ga-within-one-year" name="versioning-preview-goes-ga-within-one-year">:no_entry:</a> **DO NOT** keep a preview feature in preview for more than 1 year; it must go GA (or be removed) within 1 year after introduction.

#### Use Extensible Enums

While removing a value from an enum is a breaking change, adding value to an enum can be handled with an _extensible enum_.  An extensible enum is a string value that has been marked with a special marker - setting `modelAsString` to true within an `x-ms-enum` block.  For example:

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

<a href="#versioning-use-extensible-enums" name="versioning-use-extensible-enums">:ballot_box_with_check:</a> **YOU SHOULD** use extensible enums unless you are positive that the symbol set will **NEVER** change over time.

<a href="#deprecation" name="deprecation"></a>
### Deprecating Behavior Notification

When the [API Versioning](#api-versioning) guidance above cannot be followed and the [Azure Breaking Change Reviewers](mailto:azbreakchangereview@microsoft.com) approve a breaking change to a specific API version it must be communicated to its callers. The API version that is being deprecated must add the `azure-deprecating` response header with a semicolon-delimited string notifying the caller what is being deprecated, when it will no longer function, and a URL linking to more information such as what new operation they should use instead.

The purpose is to inform customers (when debugging/logging responses) that they must take action to modify their call to the service's operation and use a newer API version or their call will soon stop working entirely. It is not expected that client code will examine/parse this header's value in any way; it is purely informational to a human being. The string is _not_ part of an API contract (except for the semi-colon delimiters) and may be changed/improved at any time without incurring a breaking change.

<a href="#deprecation-header" name="deprecation-header">:white_check_mark:</a> **DO** include the `azure-deprecating` header in the operation's response _only if_ the operation will stop working in the future and the client _must take_ action in order for it to keep working.
> NOTE: We do not want to scare customers with this header.

<a href="#deprecation-header-value" name="deprecation-header-value">:white_check_mark:</a> **DO** make the header's value a semicolon-delimited string indicating a set of deprecations where each one indicates what is deprecating, when it is deprecating, and a URL to more information.

Deprecations should use the following pattern:
```text
<description> will retire on <date> (<url>)
```

Multiple deprecations are allowed, semicolon delimited.

Where the following placeholders should be provided:
- `description`: a human-readable description of what is being deprecated
- `date`: the target date that this will be deprecated. This should be expressed following the format in [ISO 8601](https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.1.1), e.g. "2022-10-31".
- `url`: a fully qualified url that the user can follow to learn more about what is being deprecated, preferably to Azure Updates.

For example:
- `azure-deprecating: API version 2009-27-07 will retire on 2022-12-01 (https://azure.microsoft.com/updates/video-analyzer-retirement);TLS 1.0 & 1.1 will retire on 2020-10-30 (https://azure.microsoft.com/updates/azure-active-directory-registration-service-is-ending-support-for-tls-10-and-11/)`
- `azure-deprecating: Model version 2021-01-15 used in Sentiment analysis will retire on 2022-12-01 (https://aka.ms/ta-modelversions?sentimentAnalysis)`
- `azure-deprecating: TLS 1.0 & 1.1 support will retire on 2022-10-01 (https://devblogs.microsoft.com/devops/deprecating-weak-cryptographic-standards-tls-1-0-and-1-1-in-azure-devops-services/)`

<a href="#deprecation-header-review" name="deprecation-header-review">:no_entry:</a> **DO NOT** introduce this header without approval from [Azure Breaking Change Reviewers](mailto:azbreakchangereview@microsoft.com) and an official deprecation notice on [Azure Updates](https://azure.microsoft.com/updates/).

<a href="#repeatability" name="repeatability"></a>
### Repeatability of requests

Fault tolerant applications require that clients retry requests for which they never got a response, and services must handle these retried requests idempotently. In Azure, all HTTP operations are naturally idempotent except for POST used to create a resource and [POST when used to invoke an action](
https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md#performing-an-action).

<a href="#repeatability-headers" name="repeatability-headers">:ballot_box_with_check:</a> **YOU SHOULD** support repeatable requests as defined in [OASIS Repeatable Requests Version 1.0](https://docs.oasis-open.org/odata/repeatable-requests/v1.0/repeatable-requests-v1.0.html) for POST operations to make them retriable.
- The tracked time window (difference between the `Repeatability-First-Sent` value and the current time) **MUST** be at least 5 minutes.
- Document the POST operation's support for the `Repeatability-First-Sent`, `Repeatability-Request-ID`, and `Repeatability-Result` headers in the API contract and documentation.
- Any operation that does not support repeatability headers should return a 501 (Not Implemented) response for any request that contains valid repeatability request headers.

### Long-Running Operations & Jobs

A _long-running operation (LRO)_ is typically an operation that should execute synchronously, but due to services not wanting to maintain long-lived connections (>1 seconds) and load-balancer timeouts, the operation must execute asynchronously. For this pattern, the client initiates the operation on the service, and then the client repeatedly polls the service (via another API call) to track the operation's progress/completion.

LROs are always started by 1 logical client and may be polled (have their status checked) by the same client, another client, or even multiple clients/browsers. An example would be a dashboard or portal that shows all the operations along with their status.  See the [Long Running Operations section](./ConsiderationsForServiceDesign.md#long-running-operations) in Considerations for Service Design for an introduction to the design of long-running operations.

<a href="#lro-response-time" name="lro-response-time">:white_check_mark:</a> **DO** implement an operation as an LRO if the 99th percentile response time is greater than 1 second and when the client should poll the operation before making more progress.

<a href="#lro-no-patch-lro" name="lro-no-patch-lro">:no_entry:</a> **DO NOT** implement PATCH as an LRO. If LRO update semantics are required, implement it using the [LRO POST action pattern](#lro-existing-resource) .

#### Patterns to Initiate a Long-Running Operation

<a href="#lro-valid-inputs-synchronously" name="lro-valid-inputs-synchronously">:white_check_mark:</a> **DO** perform as much validation as practical when initiating an LRO operation to alert clients of errors early.

<a href="#lro-returns-operation-location" name="lro-returns-operation-location">:white_check_mark:</a> **DO** include an `operation-location` response header with the absolute URL of the status monitor for the operation.

<a href="#lro-operation-location-includes-api-version" name="lro-operation-location-includes-api-version">:ballot_box_with_check:</a> **YOU SHOULD** include the `api-version` query parameter in the `operation-location` response header with the same version passed on the initial request but expect a client to change the `api-version` value to whatever a new/different client desires it to be.

<a href="#lro-put-response-headers" name="lro-put-response-headers">:white_check_mark:</a> **DO** include response headers with any additional values needed for a [GET polling request](#lro-poll) to the status monitor (e.g. location).

#### Create or replace operation with additional long-running processing
<a href="#put-operation-with-additional-long-running-processing" name="put-operation-with-additional-long-running-processing"></a>

<a href="#lro-create-init" name="lro-create-init">:white_check_mark:</a> **DO** use the following pattern when implementing an operation that creates or replaces a resource that involves additional long-running processing:

```text
PUT /UrlToResourceBeingCreated?api-version=<api-version>
operation-id: <optionalStatusMonitorResourceId>

<JSON Resource in body>
```

The response must look like this:

```text
201 Created
operation-id: <statusMonitorResourceId>
operation-location: https://operations/<operation-id>?api-version=<api-version>

<JSON Resource in body>
```

The request and response body schemas must be identical and represent the resource.

The PUT creates or replaces the resource immediately and returns but the additional long-running processing can take time to complete.

For an idempotent PUT (same `operation-id` or same request body within some short time window), the service should return the same response as shown above.

For a non-idempotent PUT, the service can choose to overwrite the existing resource (as if the resource were deleted) or the service can return `409-Conflict` with the error's code property indicated why this PUT operation failed.

<a href="#lro-put-operation-id-request-header" name="lro-put-operation-id-request-header">:white_check_mark:</a> **DO** allow the client to pass an `Operation-Id` header with a ID for the status monitor for the operation.

If the `Operation-Id` header is not specified, the service may create an operation-id (typically a GUID) and return it via the `operation-id` and `operation-location` response headers; in this case the service must figure out how to deal with retries/idempotency.

<a href="#lro-put-operation-id-default-is-guid" name="lro-put-operation-id-default-is-guid">:white_check_mark:</a> **DO** generate an ID (typically a GUID) for the status monitor if the `Operation-Id` header was not passed by the client.

<a href="#lro-put-operation-id-unique-except-retries" name="lro-put-operation-id-unique-except-retries">:white_check_mark:</a> **DO** fail a request with a `409-Conflict` if the `Operation-Id` header matches an existing operation unless the request is identical to the prior request (a retry scenario).

<a href="#lro-put-valid-inputs-synchronously" name="lro-put-valid-inputs-synchronously">:white_check_mark:</a> **DO** perform as much validation as practical when initiating the operation to alert clients of errors early.

<a href="#lro-put-returns-200-or-201" name="lro-put-returns-200-or-201">:white_check_mark:</a> **DO** return a `201-Created` status code for create or `200-OK` for replace from the initial request with a representation of the resource, if the resource was created or replaced successfully.

<a href="#lro-put-returns-operation-id-header" name="lro-put-returns-operation-id-header">:white_check_mark:</a> **DO** include an `Operation-Id` header in the response with the ID of the status monitor for the operation.

<a href="#lro-put-returns-operation-location" name="lro-put-returns-operation-location">:ballot_box_with_check:</a> **YOU SHOULD** include an `Operation-Location` header in the response with the absolute URL of the status monitor for the operation.

<a href="#lro-put-operation-location-includes-api-version" name="lro-put-operation-location-includes-api-version">:ballot_box_with_check:</a> **YOU SHOULD** include the `api-version` query parameter in the `Operation-Location` header with the same version passed on the initial request.

#### DELETE LRO pattern

<a href="#lro-delete" name="lro-delete">:white_check_mark:</a> **DO** use the following pattern when implementing an LRO operation to delete a resource:

```text
DELETE /UrlToResourceBeingDeleted?api-version=<api-version>
operation-id: <optionalStatusMonitorResourceId>
```

The response must look like this:

```text
202 Accepted
operation-id: <statusMonitorResourceId>
operation-location: https://operations/<operation-id>
```

Consistent with non-LRO DELETE operations, if a request body is specified, return `400-Bad Request`.

<a href="#lro-delete-operation-id-request-header" name="lro-delete-operation-id-request-header">:white_check_mark:</a> **DO** allow the client to pass an `Operation-Id` header with an ID for the operation's status monitor.

<a href="#lro-delete-operation-id-default-is-guid" name="lro-delete-operation-id-default-is-guid">:white_check_mark:</a> **DO** generate an ID (typically a GUID) for the status monitor if the `Operation-Id` header was not passed by the client.

<a href="#lro-delete-returns-202" name="lro-delete-returns-202">:white_check_mark:</a> **DO** return a `202-Accepted` status code from the request that initiates an LRO if the processing of the operation was successfully initiated.

<a href="#lro-delete-returns-only-202" name="lro-delete-returns-only-202">:warning:</a> **YOU SHOULD NOT** return any other `2xx` status code from the initial request of an LRO -- return `202-Accepted` and a status monitor even if processing was completed before the initiating request returns.

#### LRO action on a resource pattern
<a href="#post-or-delete-lro-pattern" name="post-or-delete-lro-pattern"></a><!-- Preserve old header link -->

<a href="#lro-existing-resource" name="lro-existing-resource">:white_check_mark:</a> **DO** use the following pattern when implementing an LRO action operating on an existing resource:

```text
POST /UrlToExistingResource:<action>?api-version=<api-version>&<actionParamsGoHere>
operation-id: <optionalStatusMonitorResourceId>`

<JSON Action parameters can go in body if query params don't work>
```

The response must look like this:

```text
202 Accepted
operation-id: <statusMonitorResourceId>
operation-location: https://operations/<operation-id>

<JSON Status Monitor Resource in body>
```

The request body contains information to be used to execute the action.

For an idempotent POST (same `operation-id` and request body within some short time window), the service should return the same response as the initial request.

For a non-idempotent POST, the service can treat the POST operation as idempotent (if performed within a short time window) or can treat the POST operation as initiating a brand new LRO action operation.

<a href="#lro-no-post-create" name="lro-no-post-create">:no_entry:</a> **DO NOT** use a long-running POST to create a resource -- use PUT as described above.

<a href="#lro-operation-id-request-header" name="lro-operation-id-request-header">:white_check_mark:</a> **DO** allow the client to pass an `Operation-Id` header with an ID for the operation's status monitor.

<a href="#lro-operation-id-default-is-guid" name="lro-operation-id-default-is-guid">:white_check_mark:</a> **DO** generate an ID (typically a GUID) for the status monitor if the `Operation-Id` header was not passed by the client.

<a href="#lro-operation-id-unique-except-retries" name="lro-operation-id-unique-except-retries">:white_check_mark:</a> **DO** fail a request with a `409-Conflict` if the `Operation-Id` header matches an existing operation unless the request is identical to the prior request (a retry scenario).

<a href="#lro-returns-202" name="lro-returns-202">:white_check_mark:</a> **DO** return a `202-Accepted` status code from the request that initiates an LRO action on a resource if the processing of the operation was successfully initiated.

<a href="#lro-returns-only-202" name="lro-returns-only-202">:warning:</a> **YOU SHOULD NOT** return any other `2xx` status code from the initial request of an LRO -- return `202-Accepted` and a status monitor even if processing was completed before the initiating request returns.

<a href="#lro-returns-status-monitor" name="lro-returns-status-monitor">:white_check_mark:</a> **DO** return a status monitor in the response body as described in [Obtaining status and results of long-running operations](#obtaining-status-and-results-of-long-running-operations).

#### LRO action with no related resource pattern

<a href="#lro-action-no-resource" name="lro-action-no-resource">:white_check_mark:</a> **DO** use the following pattern when implementing an LRO action not related to a specific resource (such as a batch operation):

```text
PUT <operation-endpoint>/<operation-id>?api-version=<api-version>

<JSON body with parameters for the operation>>
```

The response must look like this:

```text
201 Created
operation-location: <absolute URL of status monitor>

<JSON Status Monitor Resource in body>
```

<a href="#lro-put-action-operation-endpoint" name="lro-put-action-operation-endpoint">:ballot_box_with_check:</a> **YOU SHOULD**
define a unique operation endpoint for each LRO action with no related resource.

<a href="#lro-put-action-operation-id" name="lro-put-action-operation-id-in-path">:white_check_mark:</a> **DO** require the
`Operation-Id` as the final path segment in the URL.

Note: The `operation-id` URL segment (not header) is *required*, forcing the client to specify the status monitor's resource ID
and is also used for retries/idempotency.

<a href="#lro-put-action-returns-201" name="lro-put-action-returns-201">:white_check_mark:</a> **DO** return a `201 Created` status code
with an `operation-location` response header if the LRO Action operation was accepted for processing.

<a href="#lro-put-action-returns-status-monitor" name="lro-put-action-returns-status-monitor">:white_check_mark:</a> **DO** return a
status monitor in the response body that contains the operation status, request parameters, and when the operation completes either
the operation result or error.

Note: Since all request parameters must be present in the status monitor,
the request and response body of the PUT can be defined with a single schema.

<a href="#lro-put-action-status-monitor-url" name="lro-put-action-status-monitor-url">:ballot_box_with_check:</a> **YOU SHOULD**
return the status monitor for an operation for a subsequent GET on the URL that initiates the LRO, and use this endpoint as
the status monitor URL returned in the `operation-location` response header.

#### The Status Monitor Resource

All patterns that initiate a LRO either implicitly or explicitly create a [Status Monitor resource](https://datatracker.ietf.org/doc/html/rfc7231#section-6.3.3) in the service's `operations` collection.

<a href="#lro-status-monitor-structure" name="lro-status-monitor-structure">:white_check_mark:</a> **DO** return a status monitor in the response body that conforms with the following structure:

Property | Type        | Required | Description
-------- | ----------- | :------: | -----------
`id`     | string      | true     | The unique id of the operation
`kind`   | string enum | true(*)  | The kind of operation
`status` | string enum | true     | The operation's current status: "NotStarted", "Running", "Succeeded", "Failed", and "Canceled"
`error`  | ErrorDetail |          | If `status`=="Failed", contains reason for failure
`result` | object      |          | If `status`=="Succeeded" && Action LRO (POST or PUT), contains success result if needed
additional<br/>properties | | | Additional named or dynamic properties of the operation

(*): When a status monitor endpoint supports multiple operations with different result structures or additional properties,
the status monitor **must** be polymorphic -- it **must** contain a required `kind` property that indicates the kind of long-running operation.

#### Obtaining status and results of long-running operations

<a href="#lro-poll" name="lro-poll">:white_check_mark:</a> **DO** use the following pattern to allow clients to poll the current state of a Status Monitor resource:

```text
GET <operation-endpoint>/<operation-id>?api-version=<api-version>
```

The response must look like this:

```text
200 OK
retry-after: <delay-seconds>    (if status not terminal)

<JSON Status Monitor Resource in body>
```

<a href="#lro-status-monitor-get-returns-200" name="lro-status-monitor-get-returns-200">:white_check_mark:</a> **DO** support the GET method on the status monitor endpoint that returns a `200-OK` response with the current state of the status monitor.

<a href="#lro-status-monitor-accepts-any-api-version" name="lro-status-monitor-accepts-any-api-version">:ballot_box_with_check:</a> **YOU SHOULD** allow any valid value of the `api-version` query parameter to be used in the GET operation on the status monitor.

- Note: Clients may replace the value of `api-version` in the `operation-location` URL with a value appropriate for their application. Remember that the client initiating the LRO may not be the same client polling the LRO's status.

<a href="#lro-status-monitor-includes-all-fields" name="lro-status-monitor-includes-all-fields">:white_check_mark:</a> **DO** include the `id` of the operation and any other values needed for the client to form a GET request to the status monitor (e.g. a `location` path parameter).

<a href="#lro-status-monitor-post-action-result" name="lro-status-monitor-post-action-result">:white_check_mark:</a> **DO** include the `result` property (if any) in the status monitor for a POST action-type long-running operation when the operation completes successfully.

<a href="#lro-status-monitor-no-resource-result" name="lro-status-monitor-no-resource-result">:no_entry:</a> **DO NOT** include a `result` property in the status monitor for a long-running operation that is not an action-type long-running operation.

<a href="#lro-status-monitor-retry-after" name="lro-status-monitor-retry-after">:white_check_mark:</a> **DO** include a `retry-after` header in the response if the operation is not complete. The value of this header should be an integer number of seconds that the client should wait before polling the status monitor again.

<a href="#lro-status-monitor-retention" name="lro-status-monitor-retention">:white_check_mark:</a> **DO** retain the status monitor resource for some publicly documented period of time (at least 24 hours) after the operation completes.

#### Pattern to List Status Monitors

Use the following patterns to allow clients to list Status Monitor resources.

<a href="#lro-list-status-monitors" name="lro-list-status-monitors">:ballot_box_with_check:</a>
**YOU MAY** support a GET method on any status monitor collection URL that returns a list of the status monitors in that collection.

<a href="#lro-put-action-list-status-monitors" name="lro-put-action-list-status-monitors">:ballot_box_with_check:</a>
**YOU SHOULD** support a list operation for any status monitor collection that includes status monitors for LRO Actions with no related resource.

<a href="#lro-list-status-monitors-filter" name="lro-list-status-monitors-filter">:ballot_box_with_check:</a>
**YOU SHOULD** support the `filter` query parameter on the list operation for any polymorphic status monitor collection and support filtering on the `kind` value of the status monitor.

For example, the following request should return all status monitor resources whose `kind` is either "VMInitializing" *or* "VMRebooting"
and whose status is "NotStarted" *or* "Succeeded".

```text
GET /operations?filter=(kind eq 'VMInitializing' or kind eq 'VMRebooting') and (status eq 'NotStarted' or status eq 'Succeeded')
```

<a href="#byos" name="byos"></a>
### Bring your own Storage (BYOS)

Many services need to store and retrieve data files. For this scenario, the service should not implement its own
storage APIs and should instead leverage the existing Azure Storage service. When doing this, the customer
"owns" the storage account and just tells your service to use it. Colloquially, we call this <i>Bring Your Own Storage</i> as the customer is bringing their storage account to another service. BYOS provides significant benefits to service implementors: security, performance, uptime, etc. And, of course, most Azure customers are already familiar with the Azure Storage service.

While Azure Managed Storage may be easier to get started with, as your service evolves and matures, BYOS provides the most flexibility and implementation choices. Further, when designing your APIs, be cognizant of expressing storage concepts and how clients will access your data. For example, if you are working with blobs, then you should not expose the concept of folders.

<a href="#byos-pattern" name="byos-pattern">:white_check_mark:</a> **DO** use the Bring Your Own Storage pattern.

<a href="#byos-prefix-for-folder" name="byos-prefix-for-folder">:white_check_mark:</a> **DO** use a blob prefix for a logical folder (avoid terms such as `directory`, `folder`, or `path`).

<a href="#byos-allow-container-reuse" name="byos-allow-container-reuse">:no_entry:</a> **DO NOT** require a fresh container per operation.

<a href="#byos-authorization" name="byos-authorization">:white_check_mark:</a> **DO** use managed identity and Role Based Access Control ([RBAC](https://docs.microsoft.com/azure/role-based-access-control/overview)) as the mechanism allowing customers to grant permission to their Storage account to your service.

<a href="#byos-define-rbac-roles" name="byos-define-rbac-roles">:white_check_mark:</a> **DO** Add RBAC roles for every service operation that requires accessing Storage scoped to the exact permissions.

<a href="#byos-rbac-compatibility" name="byos-rbac-compatibility">:white_check_mark:</a> **DO** Ensure that RBAC roles are backward compatible, and specifically, do not take away permissions from a role that would break the operation of the service. Any change of RBAC roles that results in a change of the service behavior is considered a breaking change.


#### Handling 'downstream' errors
It is not uncommon to rely on other services, e.g. storage, when implementing your service. Inevitably, the services you depend on will fail. In these situations, you can include the downstream error code and text in the inner-error of the response body. This provides a consistent pattern for handling errors in the services you depend upon.

<a href="#byos-include-downstream-errors" name="byos-include-downstream-errors">:white_check_mark:</a> **DO** include error from downstream services as the 'inner-error' section of the response body.

#### Working with files
Generally speaking, there are two patterns that you will encounter when working with files; single file access, and file collections.

##### Single file access
Designing an API for accessing a single file, depending on your scenario, is relatively straight forward.

<a href="#byos-sas-token" name="byos-sas-token">:heavy_check_mark:</a> **YOU MAY** use a Shared Access Signature [SAS](https://docs.microsoft.com/azure/storage/common/storage-sas-overview) to provide access to a single file. SAS is considered the minimum security for files and can be used in lieu of, or in addition to, RBAC.

<a href="#byos-http-insecure" name="byos-http-insecure">:ballot_box_with_check:</a> **YOU SHOULD** if using HTTP (not HTTPS) document to users that all information is sent over the wire in clear text.

<a href="#byos-http-status-code" name="byos-http-status-code">:white_check_mark:</a> **DO** return an HTTP status code representing the result of your service operation's behavior.

<a href="#byos-include-storage-error" name="byos-include-storage-error">:white_check_mark:</a> **DO** include the Storage error information in the 'inner-error' section of an error response if the error was the result of an internal Storage operation failure. This helps the client determine the underlying cause of the error, e.g.: a missing storage object or insufficient permissions.

<a href="#byos-support-single-object" name="byos-support-single-object">:white_check_mark:</a> **DO** allow the customer to specify a URL path to a single Storage object if your service requires access to a single file.

<a href="#byos-last-modified" name="byos-last-modified">:heavy_check_mark:</a> **YOU MAY** allow the customer to provide a [last-modified](https://datatracker.ietf.org/doc/html/rfc7232#section-2.2) timestamp (in RFC 7231 format) for read-only files. This allows the client to specify exactly which version of the files your service should use.
When reading a file, your service passes this timestamp to Azure Storage using the [if-unmodified-since](https://datatracker.ietf.org/doc/html/rfc7232#section-3.4) request header. If the Storage operation fails with 412, the Storage object was modified and your service operation should return an appropriate 4xx status code and return the Storage error in your operation's 'inner-error' (see guideline above).

<a href="#byos-folder-support" name="byos-folder-support">:white_check_mark:</a> **DO** allow the customer to specify a URL path to a logical folder (via prefix and delimiter) if your service requires access to multiple files (within this folder). For more information, see [List Blobs API](https://docs.microsoft.com/rest/api/storageservices/list-blobs)

<a href="#byos-extensions" name="byos-extensions">:heavy_check_mark:</a> **YOU MAY** offer an `extensions` field representing an array of strings indicating file extensions of desired blobs within the logical folder.

A common pattern when working with multiple files is for your service to receive requests that contain the location(s) of files to process ("input") and a location(s) to place any files that result from processing ("output"). Note: the terms "input" and "output" are just examples; use terms more appropriate to your service's domain.

For example, a service's request body to configure BYOS may look like this:

```json
{
  "input":{
    "location": "https://mycompany.blob.core.windows.net/documents/english/?<sas token>",
    "delimiter": "/",
    "extensions" : [ ".bmp", ".jpg", ".tif", ".png" ],
    "lastModified": "Wed, 21 Oct 2015 07:28:00 GMT"
  },
  "output":{
    "location": "https://mycompany.blob.core.windows.net/documents/spanish/?<sas token>",
    "delimiter":"/"
  }
}
```

Depending on the requirements of the service, there can be any number of "input" and "output" sections, including none.

<a href="#byos-location-and-delimiter" name="byos-location-and-delimiter">:white_check_mark:</a> **DO** include a JSON object that has string values for "location" and "delimiter". For "location", the customer must pass a URL to a blob prefix which represents a directory. For "delimiter", the customer must specify the delimiter character they desire to use in the location URL; typically "/" or "\".

<a href="#byos-directory-last-modified" name="byos-directory-last-modified">:heavy_check_mark:</a> **YOU MAY** support the "lastModified" field for input directories (see guideline above).

<a href="#byos-sas-for-input-location" name="byos-sas-for-input-location">:white_check_mark:</a> **DO** support a "location" URL with a container-scoped SAS that has a minimum of `listing` and `read` permissions for input directories.

<a href="#byos-sas-for-output-location" name="byos-sas-for-output-location">:white_check_mark:</a> **DO** support a "location" URL with a container-scoped SAS that has a minimum of `write` permissions for output directories.

<a href="#condreq" name="condreq"></a>
### Conditional Requests

The [HTTP Standard][] defines request headers that clients may use to specify a _precondition_
for execution of an operation. These headers allow clients to implement efficient caching mechanisms
and avoid data loss in the event of concurrent updates to a resource. The headers that specify conditional execution are `If-Match`, `If-None-Match`, `If-Modified-Since`, `If-Unmodified-Since`, and `If-Range`.

[HTTP Standard]: https://datatracker.ietf.org/doc/html/rfc9110

<!-- condreq-support-etags-consistently has been subsumed by condreq-support but we retain the anchor to avoid broken links -->
<a href="#condreq-support-etags-consistently" name="condreq-support-etags-consistently"></a>
<!-- condreq-for-read has been subsumed by condreq-support but we retain the anchor to avoid broken links -->
<a href="#condreq-for-read" name="condreq-for-read"></a>
<!-- condreq-no-pessimistic-update has been subsumed by condreq-support but we retain the anchor to avoid broken links -->
<a href="#condreq-no-pessimistic-update" name="condreq-no-pessimistic-update"></a>
<a href="#condreq-support" name="condreq-support">:white_check_mark:</a> **DO** honor any precondition headers received as part of a client request.

The HTTP Standard does not allow precondition headers to be ignored, as it can be unsafe to do so.

<a href="#condreq-unsupported-error" name="condreq-unsupported-error">:white_check_mark:</a> **DO** return the appropriate precondition failed error response if the service cannot verify the truth of the precondition.

Note: The Azure Breaking Changes review board will allow a GA service that currently ignores precondition headers to begin honoring them in a new API version without a formal breaking change notification. The potential for disruption to customer applications is low and outweighed by the value of conforming to HTTP standards.

While conditional requests can be implemented using last modified dates, entity tags ("ETags") are strongly
preferred since last modified dates cannot distinguish updates made less than a second apart.

<a href="#condreq-return-etags" name="condreq-return-etags">:ballot_box_with_check:</a> **YOU SHOULD** return an `ETag` with any operation returning the resource or part of a resource or any update of the resource (whether the resource is returned or not).

#### Conditional Request behavior

This section gives a summary of the processing to perform for precondition headers.
See the [Conditional Requests section of the HTTP Standard][] for details on how and when to evaluate these headers.

[Conditional Requests section of the HTTP Standard]: https://datatracker.ietf.org/doc/html/rfc9110#name-conditional-requests

<a href="#condreq-for-read-behavior" name="condreq-for-read-behavior">:white_check_mark:</a> **DO** adhere to the following table for processing a GET request with precondition headers:

| GET Request | Return code | Response                                    |
|:------------|:------------|:--------------------------------------------|
| ETag value = `If-None-Match` value   | `304-Not Modified` | no additional information   |
| ETag value != `If-None-Match` value  | `200-OK`           | Response body include the serialized value of the resource (typically JSON)    |

For more control over caching, please refer to the `cache-control` [HTTP header](https://developer.mozilla.org/docs/Web/HTTP/Headers/Cache-Control).

<a href="#condreq-behavior" name="condreq-behavior">:white_check_mark:</a> **DO** adhere to the following table for processing a PUT, PATCH, or DELETE request with precondition headers:

| Operation   | Header        | Value | ETag check | Return code | Response       |
|:------------|:--------------|:------|:-----------|:------------|----------------|
| PATCH / PUT | `If-None-Match` | *     | check for _any_ version of the resource ('*' is a wildcard used to match anything), if none are found, create the resource. | `200-OK` or </br> `201-Created` </br> | Response header MUST include the new `ETag` value. Response body SHOULD include the serialized value of the resource (typically JSON).  |
| PATCH / PUT | `If-None-Match` | *     | check for _any_ version of the resource, if one is found, fail the operation |  `412-Precondition Failed` | Response body SHOULD return the serialized value of the resource (typically JSON) that was passed along with the request.|
| PATCH / PUT | `If-Match` | value of ETag     | value of `If-Match` equals the latest ETag value on the server, confirming that the version of the resource is the most current | `200-OK` or </br> `201-Created` </br> | Response header MUST include the new `ETag` value. Response body SHOULD include the serialized value of the resource (typically JSON).  |
| PATCH / PUT | `If-Match` | value of ETag     | value of `If-Match` header DOES NOT equal the latest ETag value on the server, indicating a change has ocurred since after the client fetched the resource|  `412-Precondition Failed` | Response body SHOULD return the serialized value of the resource (typically JSON) that was passed along with the request.|
| DELETE      | `If-Match` | value of ETag     | value matches the latest value on the server | `204-No Content` | Response body SHOULD be empty.  |
| DELETE      | `If-Match` | value of ETag     | value does NOT match the latest value on the server | `412-Preconditioned Failed` | Response body SHOULD be empty.|

#### Computing ETags

The strategy that you use to compute the `ETag` depends on its semantic. For example, it is natural, for resources that are inherently versioned, to use the version as the value of the `ETag`. Another common strategy for determining the value of an `ETag` is to use a hash of the resource. If a resource is not versioned, and unless computing a hash is prohibitively expensive, this is the preferred mechanism.

<a href="#condreq-etag-is-hash" name="condreq-etag-is-hash">:ballot_box_with_check:</a> **YOU SHOULD** use a hash of the representation of a resource rather than a last modified/version number

While it may be tempting to use a revision/version number for the resource as the ETag, it interferes with client's ability to retry update requests. If a client sends a conditional update request, the service acts on the request, but the client never receives a response, a subsequent identical update will be seen as a conflict even though the retried request is attempting to make the same update.

<a href="#condreq-etag-hash-entire-resource" name="condreq-etag-hash-entire-resource">:ballot_box_with_check:</a> **YOU SHOULD**, if using a hash strategy, hash the entire resource.

<a href="#condreq-strong-etag-for-range-requests" name="condreq-strong-etag-for-range-requests">:ballot_box_with_check:</a> **YOU SHOULD**, if supporting range requests, use a strong ETag in order to support caching.

<a href="#condreq-timestamp-precision" name="condreq-timestamp-precision">:heavy_check_mark:</a> **YOU MAY** use or, include, a timestamp in your resource schema. If you do this, the timestamp shouldn't be returned with more than subsecond precision, and it SHOULD be consistent with the data and format returned, e.g. consistent on milliseconds.

<a href="#condreq-weak-etags-allowed" name="condreq-weak-etags-allowed">:heavy_check_mark:</a> **YOU MAY** consider Weak ETags if you have a valid scenario for distinguishing between meaningful and cosmetic changes or if it is too expensive to compute a hash.

<a href="#condreq-etag-depends-on-encoding" name="condreq-etag-depends-on-encoding">:white_check_mark:</a> **DO**, when supporting multiple representations (e.g. Content-Encodings) for the same resource, generate different ETag values for the different representations.

<a href="#substrings" name="substrings"></a>
### Returning String Offsets & Lengths (Substrings)

All string values in JSON are inherently Unicode and UTF-8 encoded, but clients written in a high-level programming language must work with strings in that language's string encoding, which may be UTF-8, UTF-16, or CodePoints (UTF-32).
When a service response includes a string offset or length value, it should specify these values in all 3 encodings to simplify client development and ensure customer success when isolating a substring.
See the [Returning String Offsets & Lengths] section in Considerations for Service Design for more detail, including an example JSON response containing string offset and length fields.

[Returning String Offsets & Lengths]: https://github.com/microsoft/api-guidelines/blob/vNext/azure/ConsiderationsForServiceDesign.md#returning-string-offsets--lengths-substrings

<a href="#substrings-return-value-for-each-encoding" name="substrings-return-value-for-each-encoding">:white_check_mark:</a> **DO** include all 3 encodings (UTF-8, UTF-16, and CodePoint) for every string offset or length value in a service response.

<a href="#substrings-return-value-structure" name="substrings-return-value-structure">:white_check_mark:</a> **DO** define every string offset or length value in a service response as an object with the following structure:

| Property    | Type    | Required | Description |
| ----------- | ------- | :------: | ----------- |
| `utf8`      | integer | true     | The offset or length of the substring in UTF-8 encoding |
| `utf16`     | integer | true     | The offset or length of the substring in UTF-16 encoding |
| `codePoint` | integer | true     | The offset or length of the substring in CodePoint encoding |

<a href="#telemetry" name="telemetry"></a>
### Distributed Tracing & Telemetry

Azure SDK client guidelines specify that client libraries must send telemetry data through the `User-Agent` header, `X-MS-UserAgent` header, and Open Telemetry.
Client libraries are required to send telemetry and distributed tracing information on every  request. Telemetry information is vital to the effective operation of your service and should be a consideration from the outset of design and implementation efforts.

<a href="#telemetry-headers" name="telemetry-headers">:white_check_mark:</a> **DO** follow the Azure SDK client guidelines for supporting telemetry headers and Open Telemetry.

<a href="#telemetry-allow-unrecognized-headers" name="telemetry-allow-unrecognized-headers">:no_entry:</a> **DO NOT** reject a call if you have custom headers you don't understand, and specifically, distributed tracing headers.

**Additional References**
- [Azure SDK client guidelines](https://azure.github.io/azure-sdk/general_azurecore.html)
- [Azure SDK User-Agent header policy](https://azure.github.io/azure-sdk/general_azurecore.html#azurecore-http-telemetry-x-ms-useragent)
- [Azure SDK Distributed tracing policy](https://azure.github.io/azure-sdk/general_azurecore.html#distributed-tracing-policy)
- [Open Telemetry](https://opentelemetry.io/)

In addition to distributed tracing, Azure also uses a set of common correlation headers:

|Name                         |Applies to|Description|
|-----------------------------|----------|-----------|
|x-ms-client-request-id       |Both      |Optional. Caller-specified value identifying the request, in the form of a GUID with no decoration such as curly braces (e.g. `x-ms-client-request-id: 9C4D50EE-2D56-4CD3-8152-34347DC9F2B0`). If the caller provides this header the service **must** include this in their log entries to facilitate correlation of log entries for a single request. Because this header can be client-generated, it should not be assumed to be unique by the service implementation.
|x-ms-request-id              |Response  |Required. Service generated correlation id identifying the request, in the form of a GUID with no decoration such as curly braces. In contrast to the the `x-ms-client-request-id`, the service **must** ensure that this value is globally unique. Services should log this value with their traces to facilitate correlation of log entries for a single request.

## Final thoughts
These guidelines describe the upfront design considerations, technology building blocks, and common patterns that Azure teams encounter when building an API for their service. There is a great deal of information in them that can be difficult to follow. Fortunately, at Microsoft, there is a team committed to ensuring your success.

The Azure REST API Stewardship board is a collection of dedicated architects that are passionate about helping Azure service teams build interfaces that are intuitive, maintainable, consistent, and most importantly, delight our customers.
Because APIs affect nearly all downstream decisions, you are encouraged to reach out to the Stewardship board early in the development process.
These architects will work with you to apply these guidelines and identify any hidden pitfalls in your design. For more information on how to part with the Stewardship board, please refer to [Considerations for Service Design](./ConsiderationsForServiceDesign.md).
