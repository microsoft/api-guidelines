# Graph API Design API Patterns

## Introduction

The Graph REST API guidelines are an extension of the [Microsoft REST API guidelines](../guidelines). Readers of this document are assumed to be also reading the [Microsoft REST API guidelines](../guidelines) and be familiar with them.  Graph guidance is a superset of the Microsoft API guidelines and services should follow them *except* where this document outlines specific differences or exceptions to those guidelines. This document does contain additional Graph-specific guidance and additional details.

The following table of contents links back to the primary guidelines where there are no differences in Graph guidelines.  Where differences exist, the section heading is **bold**.

## 2. Table of contents
<!-- TOC depthFrom:2 depthTo:4 orderedList:false updateOnSave:false withLinks:true -->

- [Microsoft REST API Guidelines Working Group](../guidelines#microsoft-rest-api-guidelines-working-group)
- [1. Abstract](../guidelines#1-abstract)
- [2. Table of contents](../guidelines#2-table-of-contents)
- [3. Introduction](../guidelines#3-introduction)
    - [3.1. Recommended reading](../guidelines#31-recommended-reading)
- [4. Interpreting the guidelines](../guidelines#4-interpreting-the-guidelines)
    - [4.1. Application of the guidelines](../guidelines#41-application-of-the-guidelines)
    - [4.2. Guidelines for existing services and versioning of services](../guidelines#42-guidelines-for-existing-services-and-versioning-of-services)
    - [4.3. Requirements language](../guidelines#43-requirements-language)
    - [4.4. License](../guidelines#44-license)
- [5. Taxonomy](../guidelines#5-taxonomy)
    - [5.1. Errors](../guidelines#51-errors)
    - [5.2. Faults](../guidelines#52-faults)
    - [5.3. Latency](../guidelines#53-latency)
    - [5.4. Time to complete](../guidelines#54-time-to-complete)
    - [5.5. Long running API faults](../guidelines#55-long-running-api-faults)
- [6. Client guidance](../guidelines#6-client-guidance)
    - [6.1. Ignore rule](../guidelines#61-ignore-rule)
    - [6.2. Variable order rule](../guidelines#62-variable-order-rule)
    - [6.3. Silent fail rule](../guidelines#63-silent-fail-rule)
- [7. Consistency fundamentals](../guidelines#7-consistency-fundamentals)
    - [7.1. URL structure](../guidelines#71-url-structure)
    - [7.2. URL length](../guidelines#72-url-length)
    - [7.3. Canonical identifier](../guidelines#73-canonical-identifier)
    - [7.4. Supported methods](../guidelines#74-supported-methods)
        - [7.4.1. POST](../guidelines#741-post)
        - [7.4.2. PATCH](../guidelines#742-patch)
        - [7.4.3. Creating resources via PATCH (UPSERT semantics)](../guidelines#743-creating-resources-via-patch-upsert-semantics)
        - [7.4.4. Options and link headers](../guidelines#744-options-and-link-headers)
    - [7.5. Standard request headers](../guidelines#75-standard-request-headers)
    - [7.6. Standard response headers](../guidelines#76-standard-response-headers)
    - [7.7. Custom headers](../guidelines#77-custom-headers)
    - [7.8. Specifying headers as query parameters](../guidelines#78-specifying-headers-as-query-parameters)
    - [7.9. PII parameters](../guidelines#79-pii-parameters)
    - [7.10. Response formats](../guidelines#710-response-formats)
        - [7.10.1. Clients-specified response format](../guidelines#7101-clients-specified-response-format)
        - [7.10.2. Error condition responses](../guidelines#7102-error-condition-responses)
    - [7.11. HTTP Status Codes](../guidelines#711-http-status-codes)
    - [7.12. Client library optional](../guidelines#712-client-library-optional)
- [8. CORS](../guidelines#8-cors)
    - [8.1. Client guidance](../guidelines#81-client-guidance)
        - [8.1.1. Avoiding preflight](../guidelines#811-avoiding-preflight)
    - [8.2. Service guidance](../guidelines#82-service-guidance)
- [9. Collections](../guidelines#9-collections)
    - [9.1. Item keys](../guidelines#91-item-keys)
    - [9.2. Serialization](../guidelines#92-serialization)
    - [9.3. Collection URL patterns](../guidelines#93-collection-url-patterns)
        - [9.3.1. Nested collections and properties](../guidelines#931-nested-collections-and-properties)
    - [9.4. Big collections](../guidelines#94-big-collections)
    - [9.5. Changing collections](../guidelines#95-changing-collections)
    - [9.6. Sorting collections](../guidelines#96-sorting-collections)
        - [9.6.1. Interpreting a sorting expression](../guidelines#961-interpreting-a-sorting-expression)
    - [9.7. Filtering](../guidelines#97-filtering)
        - [9.7.1. Filter operations](../guidelines#971-filter-operations)
        - [9.7.2. Operator examples](../guidelines#972-operator-examples)
        - [9.7.3. Operator precedence](../guidelines#973-operator-precedence)
    - [9.8. Pagination](../guidelines#98-pagination)
        - [9.8.1. Server-driven paging](../guidelines#981-server-driven-paging)
        - [9.8.2. Client-driven paging](../guidelines#982-client-driven-paging)
        - [9.8.3. Additional considerations](../guidelines#983-additional-considerations)
    - [9.9. Compound collection operations](../guidelines#99-compound-collection-operations)
- [**9a. Resource Design**](#9a-resource-design)
    - [**9a.1. Noun Resources**](#9a1-noun-resources)
    - [**9a.2. Verb Resources**](#9a2-verb-resources)
    - [**9a.3. Resource Modeling**](#9a3-resource-modeling)
- [10. Delta queries](#10-delta-queries)
    - [10.1. Delta links](../guidelines#101-delta-links)
    - [10.2. Entity representation](../guidelines#102-entity-representation)
    - [10.3. Obtaining a delta link](../guidelines#103-obtaining-a-delta-link)
    - [10.4. Contents of a delta link response](../guidelines#104-contents-of-a-delta-link-response)
    - [10.5. Using a delta link](../guidelines#105-using-a-delta-link)
- [11. JSON standardizations](../guidelines#11-json-standardizations)
    - [11.1. JSON formatting standardization for primitive types](../guidelines#111-json-formatting-standardization-for-primitive-types)
    - [11.2. Guidelines for dates and times](../guidelines#112-guidelines-for-dates-and-times)
        - [11.2.1. Producing dates](../guidelines#1121-producing-dates)
        - [11.2.2. Consuming dates](../guidelines#1122-consuming-dates)
        - [11.2.3. Compatibility](../guidelines#1123-compatibility)
    - [11.3. JSON serialization of dates and times](../guidelines#113-json-serialization-of-dates-and-times)
        - [11.3.1. The `DateLiteral` format](../guidelines#1131-the-dateliteral-format)
        - [11.3.2. Commentary on date formatting](../guidelines#1132-commentary-on-date-formatting)
    - [11.4. Durations](../guidelines#114-durations)
    - [11.5. Intervals](../guidelines#115-intervals)
    - [11.6. Repeating intervals](../guidelines#116-repeating-intervals)
    - [**11.7. Evolvable Enums**](#117-evolvable-enums)
    - [**11.8. Dictionary Types**](#118-dictionary-types)
    - [**11.9. Ommitted Properties**](#119-ommitted-properties)
- [12. Versioning](../guidelines#12-versioning)
    - [12.1. Versioning formats](../guidelines#121-versioning-formats)
        - [12.1.1. Group versioning](../guidelines#1211-group-versioning)
    - [12.2. When to version](../guidelines#122-when-to-version)
    - [12.3. Definition of a breaking change](../guidelines#123-definition-of-a-breaking-change)
- [**13. Long running operations**](#13-long-running-operations)
    - [**13.1. Resource based long running operations (RELO)**](../long-running-operations#131-resource-based-long-running-operations-relo)
    - [**13.2. Stepwise long running operations**](../long-running-operations#132-stepwise-long-running-operations)
        - [13.2.1. PUT](../guidelines#1321-put)
        - [13.2.2. POST](../guidelines#1322-post)
        - [13.2.3. POST, hybrid model](../guidelines#1323-post-hybrid-model)
        - [13.2.4. Operations resource](../guidelines#1324-operations-resource)
        - [13.2.5. Operation resource](../guidelines#1325-operation-resource)
        - [13.2.6. Operation tombstones](../guidelines#1326-operation-tombstones)
        - [13.2.7. The typical flow, polling](../guidelines#1327-the-typical-flow-polling)
        - [13.2.8. The typical flow, push notifications](../guidelines#1328-the-typical-flow-push-notifications)
        - [13.2.9. Retry-After](../guidelines#1329-retry-after)
    - [13.3. Retention policy for operation results](../guidelines#133-retention-policy-for-operation-results)
- [14. Throttling, Quotas, and Limits](../guidelines#14-throttling-quotas-and-limits)
    - [14.1. Principles](../guidelines#141-principles)
    - [14.2. Return Codes (429 vs 503)](../guidelines#142-return-codes-429-vs-503)
    - [14.3. Retry-After and RateLimit Headers](../guidelines#143-retry-after-and-ratelimit-headers)
    - [14.4. Service Guidance](../guidelines#144-service-guidance)
        - [14.4.1. Responsiveness](../guidelines#1441-responsiveness)
        - [14.4.2. Rate Limits and Quotas](../guidelines#1442-rate-limits-and-quotas)
        - [14.4.3. Overloaded services](../guidelines#1443-overloaded-services)
        - [14.4.4. Example Response](../guidelines#1444-example-response)
    - [14.5. Caller Guidance](../guidelines#145-caller-guidance)
    - [14.6. Handling callers that ignore Retry-After headers](../guidelines#146-handling-callers-that-ignore-retry-after-headers)
- [**15. Push notifications via webhooks**](#15-push-notifications-via-webhooks)
    - [15.1. Scope](../guidelines#151-scope)
    - [15.2. Principles](../guidelines#152-principles)
    - [15.3. Types of subscriptions](../guidelines#153-types-of-subscriptions)
    - [15.4. Call sequences](../guidelines#154-call-sequences)
    - [15.5. Verifying subscriptions](../guidelines#155-verifying-subscriptions)
    - [15.6. Receiving notifications](../guidelines#156-receiving-notifications)
        - [15.6.1. Notification payload](../guidelines#1561-notification-payload)
    - [15.7. Managing subscriptions programmatically](../guidelines#157-managing-subscriptions-programmatically)
        - [15.7.1. Creating subscriptions](../guidelines#1571-creating-subscriptions)
        - [15.7.2. Updating subscriptions](../guidelines#1572-updating-subscriptions)
        - [15.7.3. Deleting subscriptions](../guidelines#1573-deleting-subscriptions)
        - [15.7.4. Enumerating subscriptions](../guidelines#1574-enumerating-subscriptions)
    - [15.8. Security](../guidelines#158-security)
- [16. Unsupported requests](../guidelines#16-unsupported-requests)
    - [16.1. Essential guidance](../guidelines#161-essential-guidance)
    - [16.2. Feature allow list](../guidelines#162-feature-allow-list)
        - [16.2.1. Error response](../guidelines#1621-error-response)
- [17. Naming guidelines](../guidelines#17-naming-guidelines)
    - [17.1. Approach](../guidelines#171-approach)
    - [17.2. Casing](../guidelines#172-casing)
    - [17.3. Names to avoid](../guidelines#173-names-to-avoid)
    - [17.4. Forming compound names](../guidelines#174-forming-compound-names)
    - [17.5. Identity properties](../guidelines#175-identity-properties)
    - [17.6. Date and time properties](../guidelines#176-date-and-time-properties)
    - [17.7. Name properties](../guidelines#177-name-properties)
    - [17.8. Collections and counts](../guidelines#178-collections-and-counts)
    - [17.9. Common property names](../guidelines#179-common-property-names)
    - [**17.10. Type namespaces**](#1710-type-namespaces)
- [18. Appendix](../guidelines#18-appendix)
    - [18.1. Sequence diagram notes](../guidelines#181-sequence-diagram-notes)
        - [18.1.1. Push notifications, per user flow](../guidelines#1811-push-notifications-per-user-flow)
        - [18.1.2. Push notifications, firehose flow](../guidelines#1812-push-notifications-firehose-flow)
    - [**18.2. Additional resources**](#182-additional-resources)

<!-- /TOC -->

## Summaries of the deltas in the Microsoft Graph Design Guidelines

### 9a. Resource Design

#### 9a.1. Noun Resources

While HTTP defines no constraints on how different resources are related together, it does encourage the use of URL path segment hierarchies to convey a relationship.  In addition to the hierarchy of resources, there are also lifetime relationships between resources, the notions of [singletons, entitySets, entities, complex types and navigation properties](entity-complex) make it possible to define a set of lifetime relationships between resources.

#### 9a.2. Verb Resources

Noun-based resources are not always the best fit for meeting the requirements of a client.  There are read-only and write scenarios where a resource can be used to represent some kind of data processing operation. The terms [function and action](Functions-and-actions) are used to identify read and write operation style resources, respectively.

#### 9a.3. Resource Modeling

There are a number of principles to be aware of when modeling resources for Microsoft Graph. [Modeling variants](modeling-variants) is important when resources have have a subset of common properties and behavior.  

### 10. Deltas

The ability to track changes (pull) occuring in the data exposed by Microsoft Graph. This is addressed implementing [change tracking (aka delta query)](deltas).

### 11.7. Evolvable Enums

[Evolvable enums](evolvable-enums) enable the use of enumerations that can add new values over time without breaking clients applications.

### 11.8. Dictionary Types

### 11.9. Omitting Properties

For scenarios where the server contains business logic that determines if a property value should be returned, or not, to the client, a returned representation can be annotated to indicate where properties are [omitted](ommitting-properties).

### 13. Long running operations

Long running operations are mostly unchanged. The most significant difference is that instead of using `Operation-Location` as the header to point to the Operation, the use of the standard `Location` header is recommended. In hybrid scenarios, a `Content-Location` header can be used to indicate the URL of the created resource and the 202 response can contain a payload. Details of the diffences are described in detail in the [Long Running Operations](long-running-operations) document.

### 15. Push notifications via webHooks

To determine how the Graph docs differ than the Microsoft REST API Guidelines.
The ability to get notified (push) when a change occurs in the data exposed by Microsoft Graph. This is addressed implementing [change notifications (aka webhooks)](webhooks).

### 17.10. Type Namespaces

Microsoft Graph model types can be declared within a [type namespaces](type-namespaces) to reduce the need to prefix types with a qualifier to ensure uniqueness.

### 18.2. Additional resources

The links below provide a set of rules to think through as you design your API.

- [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines/)
- [OData Guidelines](http://www.odata.org/documentation/)
- [Microsoft Graph Documentation](https://developer.microsoft.com/en-us/graph/docs/concepts/overview)
- [Microsoft Graph Explorer](https://aka.ms/ge)
