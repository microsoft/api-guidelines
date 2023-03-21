# Microsoft Azure REST API Guidelines

<!-- cspell:ignore autorest, BYOS, etag, idempotency, maxpagesize, innererror, trippable, nextlink, condreq, etags -->
<!-- markdownlint-disable MD033 MD049 -->

<!--
Note to contributors: All guidelines now have an anchor tag to allow cross-referencing from associated tooling.
The anchor tags within a section using a common prefix to ensure uniqueness with anchor tags in other sections.
Please ensure that you add an anchor tag to any new guidelines that you add and maintain the naming convention.
-->

## History

| Date        | Notes                                                          |
| ----------- | -------------------------------------------------------------- |
| 2022-Sep-07 | Updated URL guidelines for DNS Done Right                      |                 |
| 2022-Jul-15 | Update guidance on long-running operations                     |
| 2022-May-11 | Drop guidance on version discovery                             |
| 2022-Mar-29 | Add guidelines about using durations                           |
| 2022-Mar-25 | Update guideline for date values in headers to follow RFC 7231 |
| 2022-Feb-01 | Updated error guidance                                         |
| 2021-Sep-11 | Add long-running operations guidance                           |
| 2021-Aug-06 | Updated Azure REST Guidelines per Azure API Stewardship Board. |
| 2020-Jul-31 | Added service advice for initial versions                      |
| 2020-Mar-31 | 1st public release of the Azure REST API Guidelines            |

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

*Note: If you are creating a management plane (ARM) API, please refer to the [Azure Resource Manager Resource Provider Contract](https://github.com/Azure/azure-resource-manager-rpc).*

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

<a name="http"></a>
### HTTP
Azure services must adhere to the HTTP specification, [RFC7231](https://tools.ietf.org/html/rfc7231). This section further refines and constrains how service implementors should apply the constructs defined in the HTTP specification. It is therefore, important that you have a firm understanding of the following concepts:

- [Uniform Resource Locators (URLs)](#uniform-resource-locators-urls)
- [HTTP Request / Response Pattern](#http-request--response-pattern)
- [HTTP Query Parameters and Header Values](#http-query-parameters-and-header-values)

#### Uniform Resource Locators (URLs)

A Uniform Resource Locator (URL) is how developers access the resources of your service. Ultimately, URLs are how developers form a cognitive model of your service's resources.

<a name="http-url-pattern"></a>
:white_check_mark: **DO** use this URL pattern:
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

<a name="http-url-casing"></a>
:white_check_mark: **DO** use kebab-casing (preferred) or camel-casing for URL path segments. If the segment refers to a JSON field, use camel casing.

<a name="http-url-length"></a>
:white_check_mark: **DO** return `414-URI Too Long` if a URL exceeds 2083 characters

<a name="http-url-case-sensitivity"></a>
:white_check_mark: **DO** treat service-defined URL path segments as case-sensitive. If the passed-in case doesn't match what the service expects, the request **MUST** fail with a `404-Not found` HTTP return code.

Some customer-provided path segment values may be compared case-insensitivity if the abstraction they represent is normally compared with case-insensitivity. For example, a UUID path segment of 'c55f6b35-05f6-42da-8321-2af5099bd2a2' should be treated identical to 'C55F6B35-05F6-42DA-8321-2AF5099BD2A2'

<a name="http-url-return-casing"></a>
:white_check_mark: **DO** ensure proper casing when returning a URL in an HTTP response header value or inside a JSON response body

<a name="http-url-allowed-characters"></a>
:white_check_mark: **DO** restrict the characters in service-defined path segments to `0-9  A-Z  a-z  -  .  _  ~`, with `:` allowed only as described below to designate an action operation.

<a name="http-url-allowed-characters-2"></a>
:ballot_box_with_check: **YOU SHOULD** restrict the characters allowed in user-specified path segments (i.e. path parameters values) to `0-9  A-Z  a-z  -  .  _  ~` (do not allow `:`).

<a name="http-url-should-be-readable"></a>
:ballot_box_with_check: **YOU SHOULD** keep URLs readable; if possible, avoid UUIDs & %-encoding (ex: Cádiz is %-encoded as C%C3%A1diz)

<a name="http-url-allowed-characters-3"></a>
:heavy_check_mark: **YOU MAY** use these other characters in the URL path but they will likely require %-encoding [[RFC 3986](https://datatracker.ietf.org/doc/html/rfc3986#section-2.1)]: `/  ?  #  [  ]  @  !  $  &  '  (  )  *  +  ,  ;  =`

<a name="http-direct-endpoints"></a>
:heavy_check_mark: **YOU MAY** support a direct endpoint URL for performance/routing:
```text
https://<tenant>-<service-root>.<service>.<cloud>/...
```

Examples:
- Request URL: `https://blobstore.azure.net/contoso.com/account1/container1/blob2`
- Response header ([RFC2557](https://datatracker.ietf.org/doc/html/rfc2557#section-4)): `content-location : https://contoso-dot-com-account1.blobstore.azure.net/container1/blob2`
- GUID format: `https://00000000-0000-0000-C000-000000000046-account1.blobstore.azure.net/container1/blob2`

<a name="http-url-return-consistent-form"></a>
:white_check_mark: **DO** return URLs in response headers/bodies in a consistent form regardless of the URL used to reach the resource. Either always a UUID for `<tenant>` or always a single verified domain.

<a name="http-url-parameter-values"></a>
:heavy_check_mark: **YOU MAY** use URLs as values
```text
https://api.contoso.com/items?url=https://resources.contoso.com/shoes/fancy
```

#### HTTP Request / Response Pattern
The HTTP Request / Response pattern dictates how your API behaves. For example: POST methods that create resources must be idempotent, GET method results may be cached, the If-Modified and ETag headers offer optimistic concurrency. The URL of a service, along with its request/response bodies, establishes the overall contract that developers have with your service. As a service provider, how you manage the overall request / response pattern should be one of the first implementation decisions you make.

Cloud applications embrace failure. Therefore, to enable customers to write fault-tolerant applications, _all_ service operations (including POST) **must** be idempotent. Implementing services in an idempotent manner, with an "exactly once" semantic, enables developers to retry requests without the risk of unintended consequences.

##### Exactly Once Behavior = Client Retries & Service Idempotency

<a name="http-all-methods-idempotent"></a>
:white_check_mark: **DO** ensure that _all_ HTTP methods are idempotent.

<a name="http-use-put-or-patch"></a>
:ballot_box_with_check: **YOU SHOULD** use PUT or PATCH to create a resource as these HTTP methods are easy to implement, allow the customer to name their own resource, and are idempotent.

<a name="http-post-must-be-idempotent"></a>
:heavy_check_mark: **YOU MAY** use POST to create a resource but you must make it idempotent and, of course, the response **MUST** return the URL of the created resource with a 201-Created. One way to make POST idempotent is to use the Repeatability-Request-ID & Repeatability-First-Sent headers (See [Repeatability of requests](#repeatability-of-requests)).

##### HTTP Return Codes

<a name="http-success-status-codes"></a>
:white_check_mark: **DO** adhere to the return codes in the following table when the method completes synchronously and is successful:

Method | Description | Response Status Code
-------|-------------|---------------------
PATCH  | Create/Modify the resource with JSON Merge Patch | `200-OK`, `201-Created`
PUT    | Create/Replace the _whole_ resource | `200-OK`, `201-Created`
POST   | Create new resource (ID set by service) | `201-Created` with URL of created resource
POST   | Action | `200-OK`
GET    | Read (i.e. list) a resource collection | `200-OK`
GET    | Read the resource | `200-OK`
DELETE | Remove the resource | `204-No Content`\; avoid `404-Not Found`

<a name="http-lro-status-code"></a>
:white_check_mark: **DO** return status code `202-Accepted` and follow the guidance in [Long-Running Operations & Jobs](#long-running-operations--jobs) when a PUT, POST, or DELETE method completes asynchronously.

<a name="http-method-casing"></a>
:white_check_mark: **DO** treat method names as case sensitive and should always be in uppercase

<a name="http-return-resource"></a>
:white_check_mark: **DO** return the state of the resource after a PUT, PATCH, POST, or GET operation with a `200-OK` or `201-Created`.

<a name="http-delete-returns-204"></a>
:white_check_mark: **DO** return a `204-No Content` without a resource/body for a DELETE operation (even if the URL identifies a resource that does not exist; do not return `404-Not Found`)

<a name="http-post-action-returns-200"></a>
:white_check_mark: **DO** return a `200-OK` from a POST Action. Include a body in the response, even if it has not properties, to allow properties to be added in the future if needed.

<a name="http-return-403-vs-404"></a>
:white_check_mark: **DO** return a `403-Forbidden` when the user does not have access to the resource _unless_ this would leak information about the existence of the resource that should not be revealed for security/privacy reasons, in which case the response should be `404-Not Found`. [Rationale: a `403-Forbidden` is easier to debug for customers, but should not be used if even admitting the existence of things could potentially leak customer secrets.]

<a name="http-support-optimistic-concurrency"></a>
:white_check_mark: **DO** support caching and optimistic concurrency by honoring the the `If-Match`, `If-None-Match`, if-modified-since, and if-unmodified-since request headers and by returning the ETag and last-modified response headers

#### HTTP Query Parameters and Header Values
Because information in the service URL, as well as the request / response, are strings, there must be a predictable, well-defined scheme to convert strings to their corresponding values.

<a name="http-parameter-validation"></a>
:white_check_mark: **DO** validate all query parameter and request header values and fail the operation with `400-Bad Request` if any value fails validation. Return an error response as described in the [Handling Errors](#handling-errors) section indicating what is wrong so customer can diagnose the issue and fix it themselves.

<a name="http-parameter-serialization"></a>
:white_check_mark: **DO** use the following table when translating strings:

Data type | Document that string must be
--------- | -------
Boolean   | true / false (all lowercase)
Integer   | -2<sup>53</sup>+1 to +2<sup>53</sup>-1 (for consistency with JSON limits on integers [RFC8259](https://datatracker.ietf.org/doc/html/rfc8259))
Float     | [IEEE-754 binary64](https://en.wikipedia.org/wiki/Double-precision_floating-point_format)
String    | (Un)quoted?, max length, legal characters, case-sensitive, multiple delimiter
UUID      | 123e4567-e89b-12d3-a456-426614174000 (no {}s, hyphens, case-insensitive) [RFC4122](https://datatracker.ietf.org/doc/html/rfc4122)
Date/Time (Header) | Sun, 06 Nov 1994 08:49:37 GMT [RFC7231, Section 7.1.1.1](https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.1.1)
Date/Time (Query parameter) | YYYY-MM-DDTHH:mm:ss.sssZ (with at most 3 digits of fractional seconds) [RFC3339](https://datatracker.ietf.org/doc/html/rfc3339)
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
date                | Both       | Sun, 06 Nov 1994 08:49:37 GMT (see [RFC7231, Section 7.1.1.2](https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.1.2))
_content-type_      | Both       | application/merge-patch+json
_content-length_    | Both       | 1024
_x-ms-request-id_   | Response   | 4227cdc5-9f48-4e84-921a-10967cb785a0
ETag                | Response   | "67ab43" (see [Conditional Requests](#conditional-requests))
last-modified       | Response   | Sun, 06 Nov 1994 08:49:37 GMT
_x-ms-error-code_   | Response   | (see [Handling Errors](#handling-errors))
_azure-deprecating_ | Response   | (see [Deprecating Behavior Notification](#deprecating-behavior-notification))
retry-after         | Response   | 180 (see [RFC 7231, Section 7.1.3](https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.3))

<a name="http-header-support-standard-headers"></a>
:white_check_mark: **DO** support all headers shown in _italics_

<a name="http-header-names-casing"></a>
:white_check_mark: **DO** specify headers using kebab-casing

<a name="http-header-names-case-sensitivity"></a>
:white_check_mark: **DO** compare request header names using case-insensitivity

<a name="http-header-values-case-sensitivity"></a>
:white_check_mark: **DO** compare request header values using case-sensitivity if the header name requires it

<a name="http-header-date-values"></a>
:white_check_mark: **DO** accept date values in headers in HTTP-Date format and return date values in headers in the IMF-fixdate format as defined in [RFC7231, Section 7.1.1.1](https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.1.1), e.g. "Sun, 06 Nov 1994 08:49:37 GMT".

Note: The RFC 7321 IMF-fixdate format is a "fixed-length and single-zone subset" of the RFC 1123 / RFC 5822 format, which means: a) year must be four digits, b) the seconds component of time is required, and c) the timezone must be GMT.

<a name="http-header-request-id"></a>
:white_check_mark: **DO** create an opaque value that uniquely identifies the request and return this value in the `x-ms-request-id` response header.

Your service should include the `x-ms-request-id` value in error logs so that users can submit support requests for specific failures using this value.

<a name="http-allow-unrecognized-headers"></a>
:no_entry: **DO NOT** fail a request that contains an unrecognized header. Headers may be added by API gateways or middleware and this must be tolerated

<a name="http-no-x-custom-headers"></a>
:no_entry: **DO NOT** use "x-" prefix for custom headers, unless the header already exists in production [[RFC 6648](https://datatracker.ietf.org/doc/html/rfc6648)].

**Additional References**
- [StackOverflow - Difference between http parameters and http headers](https://stackoverflow.com/questions/40492782)
- [Standard HTTP Headers](https://httpwg.org/specs/rfc7231.html#header.field.registration)
- [Why isn't HTTP PUT allowed to do partial updates in a REST API?](https://stackoverflow.com/questions/19732423/why-isnt-http-put-allowed-to-do-partial-updates-in-a-rest-api)

<a name="rest"></a>
### REpresentational State Transfer (REST)
REST is an architectural style with broad reach that emphasizes scalability, generality, independent deployment, reduced latency via caching, and security. When applying REST to your API, you define your service’s resources as a collections of items.
These are typically the nouns you use in the vocabulary of your service. Your service's [URLs](#uniform-resource-locators-urls) determine the hierarchical path developers use to perform CRUD (create, read, update, and delete) operations on resources. Note, it's important to model resource state, not behavior.
There are patterns, later in these guidelines, that describe how to invoke behavior on your service. See [this article in the Azure Architecture Center](https://docs.microsoft.com/azure/architecture/best-practices/api-design) for a more detailed discussion of REST API design patterns.

When designing your service, it is important to optimize for the developer using your API.

<a name="rest-clear-naming"></a>
:white_check_mark: **DO** focus heavily on clear & consistent naming

<a name="rest-paths-make-sense"></a>
:white_check_mark: **DO** ensure your resource paths make sense

<a name="rest-simplify-operations"></a>
:white_check_mark: **DO** simplify operations with few required query parameters & JSON fields

<a name="rest-specify-string-value-constraints"></a>
:white_check_mark: **DO** establish clear contracts for string values

<a name="rest-use-standard-status-codes"></a>
:white_check_mark: **DO** use proper response codes/bodies so customer can diagnose their own problems and fix them without contacting Azure support or the service team

#### Resource Schema & Field Mutability

<a name="rest-response-body-is-resource-schema"></a>
:white_check_mark: **DO** use the same JSON schema for PUT request/response, PATCH response, GET response, and POST request/response on a given URL path. The PATCH request schema should contain all the same fields with no required fields. This allows one SDK type for input/output operations and enables the response to be passed back in a request.

<a name="rest-field-mutability"></a>
:white_check_mark: **DO** think about your resource's fields and how they are used:

Field Mutability | Service Request's behavior for this field
-----------------| -----------------------------------------
**Create** | Service honors field only when creating a resource. Minimize create-only fields so customers don't have to delete & re-create the resource.
**Update** | Service honors field when creating or updating a resource
**Read**   | Service returns this field in a response. If the client passed a read-only field, the service **MUST** fail the request unless the passed-in value matches the resource's current value

In addition to the above, a field may be "required" or "optional". A required field is guaranteed to always exist and will typically _not_ become a nullable field in a SDK's data structure. This allows customers to write code without performing a null-check.
Because of this, required fields can only be introduced in the 1st version of a service; it is a breaking change to introduce required fields in a later version. In addition, it is a breaking change to remove a required field or make an optional field required or vice versa.

<a name="rest-flat-is-better-than-nested"></a>
:white_check_mark: **DO** make fields simple and maintain a shallow hierarchy.

<a name="rest-get-returns-json-body"></a>
:white_check_mark: **DO** use GET for resource retrieval and return JSON in the response body

<a name="rest-patch-use-merge-patch"></a>
:white_check_mark: **DO** create and update resources using PATCH [RFC5789] with JSON Merge Patch [(RFC7396)](https://datatracker.ietf.org/doc/html/rfc7396) request body.

<a name="rest-put-for-create-or-replace"></a>
:white_check_mark: **DO** use PUT with JSON for wholesale create/replace operations. **NOTE:** If a v1 client PUTs a resource; any fields introduced in V2+ should be reset to their default values (the equivalent to DELETE followed by PUT).

<a name="rest-delete-resource"></a>
:white_check_mark: **DO** use DELETE to remove a resource.

<a name="rest-fail-for-unknown-fields"></a>
:white_check_mark: **DO** fail an operation with `400-Bad Request` if the request is improperly-formed or if any JSON field name or value is not fully understood by the specific version of the service. Return an error response as described in [Handling errors](#handling-errors) indicating what is wrong so customer can diagnose the issue and fix it themselves.

<a name="rest-secrets-allowed-in-post-response"></a>
:heavy_check_mark: **YOU MAY** return secret fields via POST **if absolutely necessary**.

<a name="rest-no-secrets-in-get-response"></a>
:no_entry: **DO NOT** return secret fields via GET. For example, do not return `administratorPassword` in JSON.

<a name="rest-no-computable-fields"></a>
:no_entry: **DO NOT** add fields to the JSON if the value is easily computable from other fields to avoid bloating the body.

##### Create / Update / Replace Processing Rules

<a name="rest-put-patch-status-codes"></a>
:white_check_mark: **DO** follow the processing below to create/update/replace a resource:

When using this method | if this condition happens | use&nbsp;this&nbsp;response&nbsp;code
---------------------- | ------------------------- | ----------------------
PATCH/PUT | Any JSON field name/value not known/valid | `400-Bad Request`
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

<a name="rest-error-code-header"></a>
:white_check_mark: **DO** return an `x-ms-error-code` response header with a string error code indicating what went wrong.

*NOTE: `x-ms-error-code` values are part of your API contract (because customer code is likely to do comparisons against them) and cannot change in the future.*

<a name="rest-error-code-enum"></a>
:heavy_check_mark: **YOU MAY** implement the `x-ms-error-code` values as an enum with `"modelAsString": true` because it's possible add new values over time.  In particular, it's only a breaking change if the same conditions result in a *different* top-level error code.

<a name="rest-add-codes-in-new-api-version"></a>
:warning: **YOU SHOULD NOT** add new top-level error codes to an existing API without bumping the service version.

<a name="rest-descriptive-error-code-values"></a>
:white_check_mark: **DO** carefully craft unique `x-ms-error-code` string values for errors that are recoverable at runtime.  Reuse common error codes for usage errors that are not recoverable.

<a name="rest-error-code-grouping"></a>
:heavy_check_mark: **YOU MAY** group common customer code errors into a few `x-ms-error-code` string values.

<a name="rest-error-code-header-and-body-match"></a>
:white_check_mark: **DO** ensure that the top-level error's `code` value is identical to the `x-ms-error-code` header's value.

<a name="rest-error-response-body-structure"></a>
:white_check_mark: **DO** provide a response body with the following structure:

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

<a name="rest-document-error-code-values"></a>
:white_check_mark: **DO** document the service's top-level error code strings; they are part of the API contract.

<a name="rest-error-non-api-contract-fields"></a>
:heavy_check_mark: **YOU MAY** treat the other fields as you wish as they are _not_ considered part of your service's API contract and customers should not take a dependency on them or their value. They exist to help customers self-diagnose issues.

<a name="rest-error-additional-properties-allowed"></a>
:heavy_check_mark: **YOU MAY** add additional properties for any data values in your error message so customers don't resort to parsing your error message.  For example, an error with `"message": "A maximum of 16 keys are allowed per account."` might also add a `"maximumKeys": 16` property.  This is not part of your API contract and should only be used for diagnosing problems.

*Note: Do not use this mechanism to provide information developers need to rely on in code (ex: the error message can give details about why you've been throttled, but the `Retry-After` should be what developers rely on to back off).*

<a name="rest-error-use-default-response"></a>
:warning: **YOU SHOULD NOT** document specific error status codes in your OpenAPI/Swagger spec unless the "default" response cannot properly describe the specific error response (e.g. body schema is different).

<a name="json"></a>
### JSON

<a name="json-field-name-casing"></a>
:white_check_mark: **DO** use camel case for all JSON field names. Do not upper-case acronyms; use camel case.

<a name="json-field-names-case-sensitivity"></a>
:white_check_mark: **DO** treat JSON field names with case-sensitivity.

<a name="json-field-values-case-sensitivity"></a>
:white_check_mark: **DO** treat JSON field values with case-sensitivity. There may be some exceptions (e.g. GUIDs) but avoid if at all possible.

Services, and the clients that access them, may be written in multiple languages. To ensure interoperability, JSON establishes the "lowest common denominator" type system, which is always sent over the wire as UTF-8 bytes. This system is very simple and consists of three types:

 Type | Description
 ---- | -----------
 Boolean | true/false (always lowercase)
 Number  | Signed floating point (IEEE-754 binary64; int range: -2<sup>53</sup>+1 to +2<sup>53</sup>-1)
 String  | Used for everything else

<a name="json-integer-values"></a>
:white_check_mark: **DO** use integers within the acceptable range of JSON number.

<a name="json-specify-string-constraints"></a>
:white_check_mark: **DO** establish a well-defined contract for the format of strings. For example, determine maximum length, legal characters, case-(in)sensitive comparisons, etc. Where possible, use standard formats, e.g. RFC3339 for date/time.

<a name="json-use-standard-string-formats"></a>
:white_check_mark: **DO** use strings formats that are well-known and easily parsable/formattable by many programming languages, e.g. RFC3339 for date/time.

<a name="json-should-be-round-trippable"></a>
:white_check_mark: **DO** ensure that information exchanged between your service and any client is "round-trippable" across multiple programming languages.

<a name="json-date-time-is-rfc3339"></a>
:white_check_mark: **DO** use [RFC3339](https://datatracker.ietf.org/doc/html/rfc3339) for date/time.

<a name="json-durations-use-fixed-time-intervals"></a>
:white_check_mark: **DO** use a fixed time interval to express durations e.g., milliseconds, seconds, minutes, days, etc., and include the time unit in the property name e.g., `backupTimeInMinutes` or `ttlSeconds`.

<a name="json-rfc3339-time-intervals-allowed"></a>
:heavy_check_mark: **YOU MAY** use [RFC3339 time intervals](https://wikipedia.org/wiki/ISO_8601#Durations) only when users must be able to specify a time interval that may change from month to month or year to year e.g., "P3M" represents 3 months no matter how many days between the start and end dates, or "P1Y" represents 366 days on a leap year. The value must be round-trippable.

<a name="json-uuid-is-rfc4412"></a>
:white_check_mark: **DO** use [RFC4122](https://datatracker.ietf.org/doc/html/rfc4122) for UUIDs.

<a name="json-may-nest-for-grouping"></a>
:heavy_check_mark: **YOU MAY** use JSON objects to group sub-fields together.

<a name="json-use-arrays-for-ordering"></a>
:heavy_check_mark: **YOU MAY** use JSON arrays if maintaining an order of values is required. Avoid arrays in other situations since arrays can be difficult and inefficient to work with, especially with JSON Merge Patch where the entire array needs to be read prior to any operation being applied to it.

<a name="json-prefer-objects-over-arrays"></a>
:ballot_box_with_check: **YOU SHOULD** use JSON objects instead of arrays whenever possible.

#### Enums & SDKs (Client libraries)
It is common for strings to have an explicit set of values. These are often reflected in the OpenAPI definition as enumerations. These are extremely useful for developer tooling, e.g. code completion, and client library generation.

However, it is not uncommon for the set of values to grow over the life of a service. For this reason, Microsoft's tooling uses the concept of an "extensible enum," which indicates that the set of values should be treated as only a _partial_ list.
This indicates to client libraries and customers that values of the enumeration field should be effectively treated as strings and that undocumented value may returned in the future. This enables the set of values to grow over time while ensuring stability in client libraries and customer code.

<a name="json-use-extensible-enums"></a>
:ballot_box_with_check: **YOU SHOULD** use extensible enumerations unless you are positive that the symbol set will NEVER change over time.

<a name="json-document-extensible-enums"></a>
:white_check_mark: **DO** document to customers that new values may appear in the future so that customers write their code today expecting these new values tomorrow.

<a name="json-return-extensible-enum-value"></a>
:heavy_check_mark: **YOU MAY** return a value for an extensible enum that is not one of the values defined for the api-version specified in the request.

<a name="json-accept-extensible-enum-value"></a>
:warning: **YOU SHOULD NOT** accept a value for an extensible enum that is not one of the values defined for the api-version specified in the request.

<a name="json-removing-enum-value-is-breaking"></a>
:no_entry: **DO NOT** remove values from your enumeration list as this breaks customer code.

#### Polymorphic types

<a name="json-avoid-polymorphism"></a>
:warning: **YOU SHOULD NOT** use polymorphic JSON types because they greatly complicate the customer code due to runtime dynamic casts and the introduction of new types in the future.

If you can't avoid them, then follow the guideline below.

<a name="json-use-discriminator-for-polymorphism"></a>
:white_check_mark: **DO** define a discriminator field indicating the kind of the resource and include any kind-specific fields in the body.

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

<a name="json-polymorphism-kind-extensible"></a>
:ballot_box_with_check: **YOU SHOULD** define the discriminator field of a polymorphic type to be an extensible enum.

**WE SHOULD CHOOSE ONE OF THE FOLLOWING TWO OPTIONS**

<a name="json-polymorphism-kind-immutable"></a>
:warning: **YOU SHOULD NOT** allow an update (patch) to change the discriminator field of a polymorphic type.

**OR**

<a name="json-polymorphism-mutable-kind"></a>
:ballot_box_with_check: **YOU SHOULD** remove all properties specific to the old discriminator value from the resource when an update (patch) changes the discriminator field.

<a name="json-polymorphism-versioning"></a>
:warning: **YOU SHOULD NOT** return properties of a polymorphic type that are not defined for the api-version specified in the request.

<a name="json-polymorphism-arrays"></a>
:warning: **YOU SHOULD NOT** have a property of an updatable resource whose value is an array of polymorphic objects.

Updating an array property with JSON merge-patch is not version-resilient if the array contains polymorphic types.

## Common API Patterns

<a name="actions"></a>
### Performing an Action
The REST specification is used to model the state of a resource, and is primarily intended to handle CRUD (Create, Read, Update, Delete) operations. However, many services require the ability to perform an action on a resource, e.g. getting the thumbnail of an image or rebooting a VM.  It is also sometimes useful to perform an action on a collection.

<a name="actions-url-pattern-for-resource-action"></a>
:ballot_box_with_check: **YOU SHOULD** pattern your URL like this to perform an action on a resource
**URL Pattern**
```text
https://.../<resource-collection>/<resource-id>:<action>?<input parameters>
```

**Example**
```text
https://.../users/Bob:grant?access=read
```

<a name="actions-url-pattern-for-collection-action"></a>
:ballot_box_with_check: **YOU SHOULD** pattern your URL like this to perform an action on a collection
**URL Pattern**
```text
https://.../<resource-collection>:<action>?<input parameters>
```

**Example**
```text
https://.../users:grant?access=read
```

Note: To avoid potential collision of actions and resource ids, you should disallow the use of the ":" character in resource ids.

<a name="actions-use-post-method"></a>
:white_check_mark: **DO** use a POST operation for any action on a resource or collection.

<a name="actions-support-repeatability-headers"></a>
:white_check_mark: **DO** support the Repeatability-Request-ID & Repeatability-First-Sent request headers if the action needs to be idempotent if retries occur.

<a name="actions-synchronous-success-status-code"></a>
:white_check_mark: **DO** return a `200-OK` when the action completes synchronously and successfully.

<a name="actions-action-name-is-verb"></a>
:ballot_box_with_check: **YOU SHOULD** use a verb as the `<action>` component of the path.

<a name="actions-no-actions-for-crud"></a>
:no_entry: **DO NOT** use an action operation when the operation behavior could reasonably be defined as one of the standard REST Create, Read, Update, Delete, or List operations.

<a name="collections"></a>
### Collections
<a name="collections-response-is-object"></a>
:white_check_mark: **DO** structure the response to a list operation as an object with a top-level array field containing the set (or subset) of resources.

<a name="collections-support-server-driven-paging"></a>
:ballot_box_with_check: **YOU SHOULD** support paging today if there is ever a chance in the future that the number of items can grow to be very large.

NOTE: It is a breaking change to add paging in the future

<a name="collections-use-get-method"></a>
:heavy_check_mark: **YOU MAY** expose an operation that lists your resources by supporting a GET method with a URL to a resource-collection (as opposed to a resource-id).

**Example Response Body**
```json
{
    "value": [
       { "id": "Item 01", "etag": "0xabc", "price": 99.95, "sizes": null },
       { … },
       { … },
       { "id": "Item 99", "etag": "0xdef", "price": 59.99, "sizes": null }
    ],
    "nextLink": "{opaqueUrl}"
 }
```

<a name="collections-items-have-id-and-etag"></a>
:white_check_mark: **DO** include the _id_ field and _etag_ field (if supported) for each item as this allows the customer to modify the item in a future operation.

<a name="collections-document-pagination-reliability"></a>
:white_check_mark: **DO** clearly document that resources may be skipped or duplicated across pages of a paginated collection unless the operation has made special provisions to prevent this (like taking a time-expiring snapshot of the collection).

<a name="collections-include-nextlink-for-more-results"></a>
:white_check_mark: **DO** return a `nextLink` field with an absolute URL that the client can GET in order to retrieve the next page of the collection.

Note: The service is responsible for performing any URL-encoding required on the `nextLink` URL.

<a name="collections-nextlink-includes-all-query-params"></a>
:white_check_mark: **DO** include any query parameters required by the service in `nextLink`, including `api-version`.

<a name="collections-response-array-name"></a>
:ballot_box_with_check: **YOU SHOULD** use `value` as the name of the top-level array field unless a more appropriate name is available.

<a name="collections-no-nextlink-on-last-page"></a>
:no_entry: **DO NOT** return the `nextLink` field at all when returning the last page of the collection.

<a name="collections-nextlink-value-never-null"></a>
:no_entry: **DO NOT** return the `nextLink` field with a value of null.

<a name="collections-avoid-count-property"></a>
:warning: **YOU SHOULD NOT** return a `count` of all objects in the collection as this may be expensive to compute.

#### Query options

<a name="collections-query-options"></a>
:heavy_check_mark: **YOU MAY** support the following query parameters allowing customers to control the list operation:

Parameter&nbsp;name | Type | Description
------------------- | ---- | -----------
`filter`       | string            | an expression on the resource type that selects the resources to be returned
`orderby`      | string&nbsp;array | a list of expressions that specify the order of the returned resources
`skip`         | integer           | an offset into the collection of the first resource to be returned
`top`          | integer           | the maximum number of resources to return from the collection
`maxpagesize`  | integer           | the maximum number of resources to include in a single response
`select`       | string&nbsp;array | a list of field names to be returned for each resource
`expand`       | string&nbsp;array | a list of the related resources to be included in line with each resource

<a name="collections-error-on-unknown-parameter"></a>
:white_check_mark: **DO** return an error if the client specifies any parameter not supported by the service.

<a name="collections-parameter-names-case-sensitivity"></a>
:white_check_mark: **DO** treat these query parameter names as case-sensitive.

<a name="collections-select-expand-ordering"></a>
:white_check_mark: **DO** apply `select` or `expand` options after applying all the query options in the table above.

<a name="collections-query-options-ordering"></a>
:white_check_mark: **DO** apply the query options to the collection in the order shown in the table above.

<a name="collections-query-options-no-dollar-sign"></a>
:no_entry: **DO NOT** prefix any of these query parameter names with "$" (the convention in the [OData standard](http://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html#sec_QueryingCollections)).

#### `filter`

<a name="collections-filter-param"></a>
:heavy_check_mark: **YOU MAY** support `filter`ing of the results of a list operation with the `filter` query parameter.

The value of the `filter` option is an expression involving the fields of the resource that produces a Boolean value. This expression is evaluated for each resource in the collection and only items where the expression evaluates to true are included in the response.

<a name="collections-filter-behavior"></a>
:white_check_mark: **DO** omit all resources from the collection for which the `filter` expression evaluates to false or to null, or references properties that are unavailable due to permissions.

Example: return all Products whose Price is less than $10.00

```text
GET https://api.contoso.com/products?`filter`=price lt 10.00
```

##### `filter` operators

:heavy_check_mark: **YOU MAY** support the following operators in `filter` expressions:

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

<a name="collections-filter-unknown-operator"></a>
:white_check_mark: **DO** respond with an error message as defined in the [Handling Errors](#handling-errors) section if a client includes an operator in a `filter` expression that is not supported by the operation.

<a name="collections-filter-operator-ordering"></a>
:white_check_mark: **DO** use the following operator precedence for supported operators when evaluating `filter` expressions. Operators are listed by category in order of precedence from highest to lowest. Operators in the same category have equal precedence and should be evaluated left to right:

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

<a name="collections-filter-functions"></a>
:heavy_check_mark: **YOU MAY** support orderby and `filter` functions such as concat and contains. For more information, see [odata Canonical Functions](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part2-url-conventions.html#_Toc31360979).

##### Operator examples
The following examples illustrate the use and semantics of each of the logical operators.

Example: all products with a name equal to 'Milk'

```text
GET https://api.contoso.com/products?`filter`=name eq 'Milk'
```

Example: all products with a name not equal to 'Milk'

```text
GET https://api.contoso.com/products?`filter`=name ne 'Milk'
```

Example: all products with the name 'Milk' that also have a price less than 2.55:

```text
GET https://api.contoso.com/products?`filter`=name eq 'Milk' and price lt 2.55
```

Example: all products that either have the name 'Milk' or have a price less than 2.55:

```text
GET https://api.contoso.com/products?`filter`=name eq 'Milk' or price lt 2.55
```

Example: all products that have the name 'Milk' or 'Eggs' and have a price less than 2.55:

```text
GET https://api.contoso.com/products?`filter`=(name eq 'Milk' or name eq 'Eggs') and price lt 2.55
```

#### orderby

<a name="collections-orderby-param"></a>
:heavy_check_mark: **YOU MAY** support sorting of the results of a list operation with the `orderby` query parameter.
*NOTE: It is unusual for a service to support `orderby` because it is very expensive to implement as it requires sorting the entire large collection before being able to return any results.*

The value of the `orderby` parameter is a comma-separated list of expressions used to sort the items.
A special case of such an expression is a property path terminating on a primitive property.

Each expression in the `orderby` parameter value may include the suffix "asc" for ascending or "desc" for descending, separated from the expression by one or more spaces.

<a name="collections-orderby-ordering"></a>
:white_check_mark: **DO** sort the collection in ascending order on an expression if "asc" or "desc" is not specified.

<a name="collections-orderby-null-ordering"></a>
:white_check_mark: **DO** sort NULL values as "less than" non-NULL values.

<a name="collections-orderby-behavior"></a>
:white_check_mark: **DO** sort items by the result values of the first expression, and then sort items with the same value for the first expression by the result value of the second expression, and so on.

<a name="collections-orderby-inherent-sort-order"></a>
:white_check_mark: **DO** use the inherent sort order for the type of the field. For example, date-time values should be sorted chronologically and not alphabetically.

<a name="collections-orderby-unsupported-field"></a>
:white_check_mark: **DO** respond with an error message as defined in the [Handling Errors](#handling-errors) section if the client requests sorting by a field that is not supported by the operation.

For example, to return all people sorted by name in ascending order:
```text
GET https://api.contoso.com/people?orderby=name
```

For example, to return all people sorted by name in descending order and a secondary sort order of hireDate in ascending order.
```text
GET https://api.contoso.com/people?orderby=name desc,hireDate
```

Sorting MUST compose with `filter`ing such that:
```text
GET https://api.contoso.com/people?`filter`=name eq 'david'&orderby=hireDate
```
will return all people whose name is David sorted in ascending order by hireDate.

##### Considerations for sorting with pagination

<a name="collections-consistent-options-with-pagination"></a>
:white_check_mark: **DO** use the same `filter`ing options and sort order for all pages of a paginated list operation response.

##### skip
<a name="collections-skip-param-definition"></a>
:white_check_mark: **DO** define the `skip` parameter as an integer with a default and minimum value of 0.

<a name="collections-skip-param"></a>
:heavy_check_mark: **YOU MAY** allow clients to pass the `skip` query parameter to specify an offset into collection of the first resource to be returned.
##### top

<a name="collections-"></a>
<a name="collections-top-param"></a>
:heavy_check_mark: **YOU MAY** allow clients to pass the `top` query parameter to specify the maximum number of resources to return from the collection.

If supporting `top`:
:white_check_mark: **DO** define the `top` parameter as an integer with a minimum value of 1. If not specified, `top` has a default value of infinity.

<a name="collections-top-behavior"></a>
:white_check_mark: **DO** return the collection's `top` number of resources (if available), starting from `skip`.

##### maxpagesize

<a name="collections-maxpagesize-param"></a>
:heavy_check_mark: **YOU MAY** allow clients to pass the `maxpagesize` query parameter to specify the maximum number of resources to include in a single page response.

<a name="collections-maxpagesize-definition"></a>
:white_check_mark: **DO** define the `maxpagesize` parameter as an optional integer with a default value appropriate for the collection.

<a name="collections-maxpagesize-might-return-fewer"></a>
:white_check_mark: **DO** make clear in documentation of the `maxpagesize` parameter that the operation may choose to return fewer resources than the value specified.

<a name="versioning"></a>
### API Versioning

Azure services need to change over time. However, when changing a service, there are 2 requirements:
 1. Already-running customer workloads must not break due to a service change
 2. Customers can adopt a new service version without requiring any code changes (Of course, the customer must modify code to leverage any new service features.)

*NOTE: the [Azure Breaking Change Policy](http://aka.ms/AzBreakingChangesPolicy/) has tables (section 5) describing what kinds of changes are considered breaking. Breaking changes are allowable (due to security/compliance/etc.) if approved by the [Azure Breaking Change Reviewers](mailto:azbreakchangereview@microsoft.com) but only following ample communication to customers and a lengthy deprecation period.*

<a name="versioning-review-required"></a>
:white_check_mark: **DO** review any API changes with the Azure API Stewardship Board

Clients specify the version of the API to be used in every request to the service, even requests to an `Operation-Location` or `nextLink` URL returned by the service.

<a name="versioning-api-version-query-param"></a>
:white_check_mark: **DO** use a required query parameter named `api-version` on every operation for the client to specify the API version.

<a name="versioning-date-based-versioning"></a>
:white_check_mark: **DO** use `YYYY-MM-DD` date values, with a `-preview` suffix for preview versions, as the valid values for `api-version`.

```text
PUT https://service.azure.com/users/Jeff?api-version=2021-06-04
```

<a name="versioning-use-later-date"></a>
:white_check_mark: **DO** use a later date for each new preview version

When releasing a new preview, the service team may completely retire any previous preview versions after giving customers at least 90 days to upgrade their code

<a name="versioning-no-breaking-changes"></a>
:no_entry: **DO NOT** introduce any breaking changes into the service.

<a name="versioning-no-version-in-path"></a>
:no_entry: **DO NOT** include a version number segment in any operation path.

<a name="versioning-use-later-date-2"></a>
:no_entry: **DO NOT** use the same date when transitioning from a preview API to a GA API. If the preview `api-version` is '2021-06-04-preview', the GA version of the API **must be** a date later than 2021-06-04

<a name="versioning-preview-goes-ga-within-one-year"></a>
:no_entry: **DO NOT** keep a preview feature in preview for more than 1 year; it must go GA (or be removed) within 1 year after introduction.

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

<a name="versioning-use-extensible-enums"></a>
:ballot_box_with_check: **You SHOULD** use extensible enums unless you are positive that the symbol set will **NEVER** change over time.

<a name="deprecation"></a>
### Deprecating Behavior Notification

When the [API Versioning](#API-Versioning) guidance above cannot be followed and the [Azure Breaking Change Reviewers](mailto:azbreakchangereview@microsoft.com) approve a [breaking change](#123-definition-of-a-breaking-change) to a specific API version it must be communicated to its callers. The API version that is being deprecated must add the `azure-deprecating` response header with a semicolon-delimited string notifying the caller what is being deprecated, when it will no longer function, and a URL linking to more information such as what new operation they should use instead.

The purpose is to inform customers (when debugging/logging responses) that they must take action to modify their call to the service's operation and use a newer API version or their call will soon stop working entirely. It is not expected that client code will examine/parse this header's value in any way; it is purely informational to a human being. The string is _not_ part of an API contract (except for the semi-colon delimiters) and may be changed/improved at any time without incurring a breaking change.

<a name="deprecation-header"></a>
:white_check_mark: **DO** include the `azure-deprecating` header in the operation's response _only if_ the operation will stop working in the future and the client _must take_ action in order for it to keep working. 
> NOTE: We do not want to scare customers with this header.

<a name="deprecation-header-value"></a>
:white_check_mark: **DO** make the header's value a semicolon-delimited string indicating a set of deprecations where each one indicates what is deprecating, when it is deprecating, and a URL to more information.

Deprecations should use the following pattern:
```text
<description> will retire on <date> (`url`);
```

Where the following placeholders should be provided:
- `description`: a human-readable description of what is being deprecated
- `date`: the target date that this will be deprecated. This should be expressed following the format in [ISO 8601](https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.1.1), e.g. "2022-10-31".
- `url`: a fully qualified url that the user can follow to learn more about what is being deprecated, preferably to Azure Updates.

For example:
```text
azure-deprecating: API version 2009-27-07 will retire on 2022-12-01 (https://azure.microsoft.com/updates/video-analyzer-retirement);TLS 1.0 & 1.1 will retire on 2020-10-30 (https://azure.microsoft.com/updates/azure-active-directory-registration-service-is-ending-support-for-tls-10-and-11/)
```

<a name="deprecation-header-review"></a>
:no_entry: **DO NOT** introduce this header without approval from [Azure Breaking Change Reviewers](mailto:azbreakchangereview@microsoft.com) and an official deprecation notice on [Azure Updates](https://azure.microsoft.com/updates/).

<a name="repeatability"></a>
### Repeatability of requests

The ability to retry failed requests for which a client never received a response greatly simplifies the ability to write resilient distributed applications. While HTTP designates some methods as safe and/or idempotent (and thus retryable), being able to retry other operations such as create-using-POST-to-collection is desirable.

<a name="repeatability-headers"></a>
:ballot_box_with_check: **YOU SHOULD** support repeatable requests according as defined in [OASIS Repeatable Requests Version 1.0](https://docs.oasis-open.org/odata/repeatable-requests/v1.0/repeatable-requests-v1.0.html).

- The tracked time window (difference between the `Repeatability-First-Sent` value and the current time) **MUST** be at least 5 minutes.
- A service advertises support for repeatability requests by adding the `Repeatability-First-Sent` and `Repeatability-Request-ID` to the set of headers for a given operation.
- When understood, all endpoints co-located behind a DNS name **MUST** understand the header. This means that a service **MUST NOT** ignore the presence of a header for any endpoints behind the DNS name, but rather fail the request containing a `Repeatability-Request-ID` header if that particular endpoint lacks support for repeatable requests. Such partial support **SHOULD** be avoided due to the confusion it causes for clients.

<a name="lro"></a>
### Long-Running Operations & Jobs

When the processing for an operation may take a significant amount of time to complete, it should be
implemented as a _long-running operation (LRO)_. This allows clients to continue running while the
operation is being processed. The client obtains the outcome of the operation at some later time
through another API call.
See the [Long Running Operations section](./ConsiderationsForServiceDesign.md#long-running-operations) in
Considerations for Service Design for an introduction to the design of long-running operations.

<a name="lro-response-time"></a>
:white_check_mark: **DO** implement an operation as an LRO if the 99th percentile response time is greater than 1s.

<a name="lro-no-patch-lro"></a>
:no_entry: **DO NOT** implement PATCH as an LRO.  If LRO update is required it must be implemented with POST.

In rare instances where an operation may take a _very long_ time to complete, e.g. longer than 15 minutes,
it may be better to expose this as a first class resource of the API rather than as an operation on another resource.

There are two basic patterns for long-running operations in Azure. The first pattern is used for a POST and DELETE
operations that initiate the LRO. These return a `202 Accepted` response with a JSON status monitor in the response body.
The second pattern applies only in the case of a PUT operation to create a resource that also involves additional long-running processing.
For guidance on when to use a specific pattern, please refer to [Considerations for Service Design, Long Running Operations](./ConsiderationsForServiceDesign.md#long-running-operations).
These are described in the following two sections.

#### POST or DELETE LRO pattern

A POST or DELETE long-running operation accepts a request from the client to initiate the operation processing and returns
a [status monitor](https://datatracker.ietf.org/doc/html/rfc7231#section-6.3.3) that reports the operation's progress.

<a name="lro-no-post-create"></a>
:no_entry: **DO NOT** use a long-running POST to create a resource -- use PUT as described below.

<a name="lro-operation-id-request-header"></a>
:white_check_mark: **DO** allow the client to pass an `Operation-Id` header with an ID for the operation's status monitor.

<a name="lro-operation-id-default-is-guid"></a>
:white_check_mark: **DO** generate an ID (typically a GUID) for the status monitor if the `Operation-Id` header was not passed by the client.

<a name="lro-operation-id-unique-except-retries"></a>
:white_check_mark: **DO** fail a request with a `400-BadRequest` if the `Operation-Id` header matches an existing operation unless the request is identical to the prior request (a retry scenario).

<a name="lro-valid-inputs-synchronously"></a>
:white_check_mark: **DO** perform as much validation as practical when initiating the operation to alert clients of errors early.

<a name="lro-returns-202"></a>
:white_check_mark: **DO** return a `202-Accepted` status code from the request that initiates an LRO if the processing of the operation was successfully initiated (except for "PUT with additional processing" type LRO).

<a name="lro-returns-only-202"></a>
:warning: **YOU SHOULD NOT** return any other `2xx` status code from the initial request of an LRO -- return `202-Accepted` and a status monitor even if processing was completed before the initiating request returns.

<a name="lro-returns-status-monitor"></a>
:white_check_mark: **DO** return a status monitor in the response body as described in [Obtaining status and results of long-running operations](#obtaining-status-and-results-of-long-running-operations).

<a name="lro-returns-operation-location"></a>
:ballot_box_with_check: **YOU SHOULD** include an `Operation-Location` header in the response with the absolute URL of the status monitor for the operation.

<a name="lro-operation-location-includes-api-version"></a>
:ballot_box_with_check: **YOU SHOULD** include the `api-version` query parameter in the `Operation-Location` header with the same version passed on the initial request if it is required by the get operation on the status monitor.

#### PUT operation with additional long-running processing

For a PUT (create or replace) with additional long-running processing:

<a name="lro-put-operation-id-request-header"></a>
:white_check_mark: **DO** allow the client to pass an `Operation-Id` header with a ID for the status monitor for the operation.

<a name="lro-put-operation-id-default-is-guid"></a>
:white_check_mark: **DO** generate an ID (typically a GUID) for the status monitor if the `Operation-Id` header was not passed by the client.

<a name="lro-put-operation-id-unique-except-retries"></a>
:white_check_mark: **DO** fail a request with a `400-BadRequest` if the `Operation-Id` header that matches an existing operation unless the request is identical to the prior request (a retry scenario).

<a name="lro-put-valid-inputs-synchronously"></a>
:white_check_mark: **DO** perform as much validation as practical when initiating the operation to alert clients of errors early.

<a name="lro-put-returns-200-or-201"></a>
:white_check_mark: **DO** return a `201-Created` status code for create or `200-OK` for replace from the initial request with a representation of the resource if the resource was created successfully.

<a name="lro-put-returns-operation-id-header"></a>
:white_check_mark: **DO** include an `Operation-Id` header in the response with the ID of the status monitor for the operation.

<a name="lro-put-response-headers"></a>
:white_check_mark: **DO** include response headers with any additional values needed for a GET request to the status monitor (e.g. location).

<a name="lro-put-returns-operation-location"></a>
:ballot_box_with_check: **YOU SHOULD** include an `Operation-Location` header in the response with the absolute URL of the status monitor for the operation.

<a name="lro-put-operation-location-includes-api-version"></a>
:ballot_box_with_check: **YOU SHOULD** include the `api-version` query parameter in the `Operation-Location` header with the same version passed on the initial request if it is required by the get operation on the status monitor.

#### Obtaining status and results of long-running operations

For all long-running operations, the client will issue a GET on a status monitor resource to obtain the current status of the operation.

<a name="lro-status-monitor-get-returns-200"></a>
:white_check_mark: **DO** support the GET method on the status monitor endpoint that returns a `200-OK` response with the current state of the status monitor.

<a name="lro-status-monitor-accepts-any-api-version"></a>
:ballot_box_with_check: **YOU SHOULD** allow any valid value of the `api-version` query parameter to be used in the get operation on the status monitor.

Note: Clients may replace the value of `api-version` in the `Operation-Location` URI with a value appropriate for their application.

<a name="lro-status-monitor-structure"></a>
:white_check_mark: **DO** return a status monitor in the response body that conforms with the following structure:

**OperationStatus** : Object

Property | Type        | Required | Description
-------- | ----------- | :------: | -----------
`id`     | string      | true     | The unique id of the operation
`status` | string      | true     | enum that includes values "NotStarted", "Running", "Succeeded", "Failed", and "Canceled"
`error`  | ErrorDetail |          | Error object that describes the error when status is "Failed"
`result` | object      |          | Only for POST action-type LRO, the results of the operation when completed successfully
additional<br/>properties | |     | Additional named or dynamic properties of the operation

<a name="lro-status-monitor-includes-all-fields"></a>
:white_check_mark: **DO** include the `id` of the operation and any other values needed for the client to form a GET request to the status monitor (e.g. a `location` path parameter).

<a name="lro-status-monitor-retry-after"></a>
:white_check_mark: **DO** include a `Retry-After` header in the response to GET requests to the status monitor if the operation is not complete. The value of this header should be an integer number of seconds to wait before making the next request to the status monitor.

<a name="lro-status-monitor-post-action-result"></a>
:white_check_mark: **DO** include the `result` property (if any) in the status monitor for a POST action-type long-running operation when the operation completes successfully.

<a name="lro-status-monitor-no-resource-result"></a>
:no_entry: **DO NOT** include a `result` property in the status monitor for a long-running operation that is not a POST action-type long-running operation.

<a name="lro-status-monitor-retention"></a>
:white_check_mark: **DO** retain the status monitor resource for some publicly documented period of time (at least 24 hours) after the operation completes.

<a name="byos"></a>
### Bring your own Storage (BYOS)
Many services need to store and retrieve data files. For this scenario, the service should not implement its own
storage APIs and should instead leverage the existing Azure Storage service. When doing this, the customer
"owns" the storage account and just tells your service to use it. Colloquially, we call this <i>Bring Your Own Storage</i> as the customer is bringing their storage account to another service. BYOS provides significant benefits to service implementors: security, performance, uptime, etc. And, of course, most Azure customers are already familiar with the Azure Storage service.

While Azure Managed Storage may be easier to get started with, as your service evolves and matures, BYOS provides the most flexibility and implementation choices. Further, when designing your APIs, be cognizant of expressing storage concepts and how clients will access your data. For example, if you are working with blobs, then you should not expose the concept of folders.

<a name="byos-pattern"></a>
:white_check_mark: **DO** use the Bring Your Own Storage pattern.

<a name="byos-prefix-for-folder"></a>
:white_check_mark: **DO** use a blob prefix for a logical folder (avoid terms such as ```directory```, ```folder```, or ```path```).

<a name="byos-allow-container-reuse"></a>
:no_entry: **DO NOT** require a fresh container per operation.

<a name="byos-authorization"></a>
:white_check_mark: **DO** use managed identity and Role Based Access Control ([RBAC](https://docs.microsoft.com/azure/role-based-access-control/overview)) as the mechanism allowing customers to grant permission to their Storage account to your service.

<a name="byos-define-rbac-roles"></a>
:white_check_mark: **DO** Add RBAC roles for every service operation that requires accessing Storage scoped to the exact permissions.

<a name="byos-rbac-compatibility"></a>
:white_check_mark: **DO** Ensure that RBAC roles are backward compatible, and specifically, do not take away permissions from a role that would break the operation of the service. Any change of RBAC roles that results in a change of the service behavior is considered a breaking change.


#### Handling 'downstream' errors
It is not uncommon to rely on other services, e.g. storage, when implementing your service. Inevitably, the services you depend on will fail. In these situations, you can include the downstream error code and text in the inner-error of the response body. This provides a consistent pattern for handling errors in the services you depend upon.

<a name="byos-include-downstream-errors"></a>
:white_check_mark: **DO** include error from downstream services as the 'inner-error' section of the response body.

#### Working with files
Generally speaking, there are two patterns that you will encounter when working with files; single file access, and file collections.

##### Single file access
Designing an API for accessing a single file, depending on your scenario, is relatively straight forward.

<a name="byos-sas-token"></a>
:heavy_check_mark: **YOU MAY** use a Shared Access Signature [SAS](https://docs.microsoft.com/azure/storage/common/storage-sas-overview) to provide access to a single file. SAS is considered the minimum security for files and can be used in lieu of, or in addition to, RBAC.

<a name="byos-http-insecure"></a>
:ballot_box_with_check: **YOU SHOULD** if using HTTP (not HTTPS) document to users that all information is sent over the wire in clear text.

<a name="byos-http-status-code"></a>
:white_check_mark: **DO** return an HTTP status code representing the result of your service operation's behavior.

<a name="byos-include-storage-error"></a>
:white_check_mark: **DO** include the Storage error information in the 'inner-error' section of an error response if the error was the result of an internal Storage operation failure. This helps the client determine the underlying cause of the error, e.g.: a missing storage object or insufficient permissions.

<a name="byos-support-single-object"></a>
:white_check_mark: **DO** allow the customer to specify a URL path to a single Storage object if your service requires access to a single file.

<a name="byos-last-modified"></a>
:heavy_check_mark: **YOU MAY** allow the customer to provide a [last-modified](https://datatracker.ietf.org/doc/html/rfc7232#section-2.2) timestamp (in RFC1123 format) for read-only files. This allows the client to specify exactly which version of the files your service should use.
When reading a file, your service passes this timestamp to Azure Storage using the [if-unmodified-since](https://datatracker.ietf.org/doc/html/rfc7232#section-3.4) request header. If the Storage operation fails with 412, the Storage object was modified and your service operation should return an appropriate 4xx status code and return the Storage error in your operation's 'inner-error' (see guideline above).

<a name="byos-folder-support"></a>
:white_check_mark: **DO** allow the customer to specify a URL path to a logical folder (via prefix and delimiter) if your service requires access to multiple files (within this folder). For more information, see [List Blobs API](https://docs.microsoft.com/rest/api/storageservices/list-blobs)

<a name="byos-extensions"></a>
:heavy_check_mark: **YOU MAY** offer an `extensions` field representing an array of strings indicating file extensions of desired blobs within the logical folder.

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

<a name="byos-location-and-delimiter"></a>
:white_check_mark: **DO** include a JSON object that has string values for "location" and "delimiter". For "location", the customer must pass a URL to a blob prefix which represents a directory. For "delimiter", the customer must specify the delimiter character they desire to use in the location URL; typically "/" or "\".

<a name="byos-directory-last-modified"></a>
:heavy_check_mark: **YOU MAY** support the "lastModified" field for input directories (see guideline above).

<a name="byos-sas-for-input-location"></a>
:white_check_mark: **DO** support a "location" URL with a container-scoped SAS that has a minimum of `listing` and `read` permissions for input directories.

<a name="byos-sas-for-output-location"></a>
:white_check_mark: **DO** support a "location" URL with a container-scoped SAS that has a minimum of `write` permissions for output directories.

<a name="condreq"></a>
### Conditional Requests
When designing an API, you will almost certainly have to manage how your resource is updated. For example, if your resource is a bank account, you will want to ensure that one transaction--say depositing money--does not overwrite a previous transaction.
Similarly, it could be very expensive to send a resource to a client. This could be because of its size, network conditions, or a myriad of other reasons. To enable this level of control, services should leverage an `ETag` header, or "entity tag," which will identify the 'version' or 'instance' of the resource a particular client is working with.
An `ETag` is always set by the service and will enable you to _conditionally_ control how your service responds to requests, enabling you to provide predictable updates and more efficient access.

<a name="condreq-return-etags"></a>
:ballot_box_with_check: **YOU SHOULD** return an `ETag` with any operation returning the resource or part of a resource or any update of the resource (whether the resource is returned or not).

<a name="condreq-support-etags-consistently"></a>
:ballot_box_with_check: **YOU SHOULD** use `ETag`s consistently across your API, i.e. if you use an `ETag`, accept it on all other operations.

You can learn more about conditional requests by reading [RFC7232](https://datatracker.ietf.org/doc/html/rfc7232).

#### Cache Control
One of the more common uses for `ETag` headers is cache control, also referred to a "conditional GET." This is especially useful when resources are large in size, expensive to compute/calculate, or hard to reach (significant network latency). That is, using the value of the `ETag` , the server can determine if the resource has changed. If there are no changes, then there is no need to return the resource, as the client already has the most recent version.

Implementing this strategy is relatively straightforward. First, you will return an `ETag` with a value that uniquely identifies the instance (or version) of the resource. The [Computing ETags](#computing-etags) section provides guidance on how to properly calculate the value of your `ETag`.
In these scenarios, when a request is made by the client an `ETag` header is returned, with a value that uniquely identifies that specific instance (or version) of the resource. The `ETag` value can then be sent in subsequent requests as part of the `If-None-Match` header.
This tells the service to compare the `ETag` that came in with the request, with the latest value that it has calculated. If the two values are the same, then it is not necessary to return the resource to the client--it already has it. If they are different, then the service will return the latest version of the resource, along with the updated `ETag` value in the header.

<a name="condreq-for-read"></a>
:ballot_box_with_check: **YOU SHOULD** implement conditional read strategies

When supporting conditional read strategies:

<a name="condreq-for-read-behavior"></a>
:white_check_mark: **DO** adhere to the following table for guidance:

| GET Request | Return code | Response                                    |
|:------------|:------------|:--------------------------------------------|
| ETag value = `If-None-Match` value   | `304-Not Modified` | no additional information   |
| ETag value != `If-None-Match` value  | `200-OK`           | Response body include the serialized value of the resource (typically JSON)    |

For more control over caching, please refer to the `cache-control` [HTTP header](https://developer.mozilla.org/docs/Web/HTTP/Headers/Cache-Control).

#### Optimistic Concurrency
An `ETag` should also be used to reflect the create, update, and delete policies of your service. Specifically, you should avoid a "pessimistic" strategy where the 'last write always wins." These can be expensive to build and scale because avoiding the "lost update" problem often requires sophisticated concurrency controls.
Instead, implement an "optimistic concurrency" strategy, where the incoming state of the resource is first compared against what currently resides in the service. Optimistic concurrency strategies are implemented through the combination of `ETags` and the [HTTP Request / Response Pattern](#http-request--response-pattern).

<a name="condreq-no-pessimistic-update"></a>
:warning: **YOU SHOULD NOT** implement pessimistic update strategies, e.g. last writer wins.

When supporting optimistic concurrency:

<a name="condreq-behavior"></a>
:white_check_mark: **DO** adhere to the following table for guidance:

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

<a name="condreq-etag-is-hash"></a>
:ballot_box_with_check: **YOU SHOULD** use a hash of the representation of a resource rather than a last modified/version number

While it may be tempting to use a revision/version number for the resource as the ETag, it interferes with client's ability to retry update requests. If a client sends a conditional update request, the service acts on the request, but the client never receives a response, a subsequent identical update will be seen as a conflict even though the retried request is attempting to make the same update.

<a name="condreq-etag-hash-entire-resource"></a>
:ballot_box_with_check: **YOU SHOULD**, if using a hash strategy, hash the entire resource.

<a name="condreq-strong-etag-for-range-requests"></a>
:ballot_box_with_check: **YOU SHOULD**, if supporting range requests, use a strong ETag in order to support caching.

<a name="condreq-timestamp-precision"></a>
:heavy_check_mark: **YOU MAY** use or, include, a timestamp in your resource schema. If you do this, the timestamp shouldn't be returned with more than subsecond precision, and it SHOULD be consistent with the data and format returned, e.g. consistent on milliseconds.

<a name="condreq-weak-etags-allowed"></a>
:heavy_check_mark: **YOU MAY** consider Weak ETags if you have a valid scenario for distinguishing between meaningful and cosmetic changes or if it is too expensive to compute a hash.

<a name="condreq-etag-depends-on-encoding"></a>
:white_check_box: **DO**, when supporting multiple representations (e.g. Content-Encodings) for the same resource, generate different ETag values for the different representations.

<a name="telemetry"></a>
### Distributed Tracing & Telemetry
Azure SDK client guidelines specify that client libraries must send telemetry data through the `User-Agent` header, `X-MS-UserAgent` header, and Open Telemetry.
Client libraries are required to send telemetry and distributed tracing information on every  request. Telemetry information is vital to the effective operation of your service and should be a consideration from the outset of design and implementation efforts.

<a name="telemetry-headers"></a>
:white_check_mark: **DO** follow the Azure SDK client guidelines for supporting telemetry headers and Open Telemetry.

<a name="telemetry-allow-unrecognized-headers"></a>
:no_entry: **DO NOT** reject a call if you have custom headers you don't understand, and specifically, distributed tracing headers.

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
