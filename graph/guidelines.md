# Graph API Design API Patterns

## Introduction

The Graph REST API guidelines are an extension of the [Microsoft REST API guidelines](../guidelines). Readers of this document are assumed to be also reading the [Microsoft REST API guidelines](../guidelines) and be familiar with them.  Graph guidance is a superset of the Microsoft API guidelines and services should follow them *except* where this document outlines specific differences or exceptions to those guidelines. This document does contain additional Graph-specific guidance and additional details.

The following table of contents links back to the primary guidelines where there are no differences in Graph guidelines.  Where differences exist, the section heading is **bold**.

## 2. Table of contents
<!-- TOC depthFrom:2 depthTo:4 orderedList:false updateOnSave:false withLinks:true -->

- [Microsoft REST API Guidelines Working Group](../Guidelines.md#microsoft-rest-api-guidelines-working-group)
- [1. Abstract](../Guidelines.md#1-abstract)
- [2. Table of contents](../Guidelines.md#2-table-of-contents)
- [3. Introduction](../Guidelines.md#3-introduction)
    - [3.1. Recommended reading](../Guidelines.md#31-recommended-reading)
- [4. Interpreting the guidelines](../Guidelines.md#4-interpreting-the-guidelines)
    - [4.1. Application of the guidelines](../Guidelines.md#41-application-of-the-guidelines)
    - [4.2. Guidelines for existing services and versioning of services](../Guidelines.md#42-guidelines-for-existing-services-and-versioning-of-services)
    - [4.3. Requirements language](../Guidelines.md#43-requirements-language)
    - [4.4. License](../Guidelines.md#44-license)
- [5. Taxonomy](../Guidelines.md#5-taxonomy)
    - [5.1. Errors](../Guidelines.md#51-errors)
    - [5.2. Faults](../Guidelines.md#52-faults)
    - [5.3. Latency](../Guidelines.md#53-latency)
    - [5.4. Time to complete](../Guidelines.md#54-time-to-complete)
    - [5.5. Long running API faults](../Guidelines.md#55-long-running-api-faults)
- [6. Client guidance](../Guidelines.md#6-client-guidance)
    - [6.1. Ignore rule](../Guidelines.md#61-ignore-rule)
    - [6.2. Variable order rule](../Guidelines.md#62-variable-order-rule)
    - [6.3. Silent fail rule](../Guidelines.md#63-silent-fail-rule)
- [7. Consistency fundamentals](../Guidelines.md#7-consistency-fundamentals)
    - [7.1. URL structure](../Guidelines.md#71-url-structure)
    - [7.2. URL length](../Guidelines.md#72-url-length)
    - [7.3. Canonical identifier](../Guidelines.md#73-canonical-identifier)
    - [7.4. Supported methods](../Guidelines.md#74-supported-methods)
        - [7.4.1. POST](../Guidelines.md#741-post)
        - [7.4.2. PATCH](../Guidelines.md#742-patch)
        - [7.4.3. Creating resources via PATCH (UPSERT semantics)](../Guidelines.md#743-creating-resources-via-patch-upsert-semantics)
        - [7.4.4. Options and link headers](../Guidelines.md#744-options-and-link-headers)
    - [7.5. Standard request headers](../Guidelines.md#75-standard-request-headers)
    - [7.6. Standard response headers](../Guidelines.md#76-standard-response-headers)
    - [7.7. Custom headers](../Guidelines.md#77-custom-headers)
    - [7.8. Specifying headers as query parameters](../Guidelines.md#78-specifying-headers-as-query-parameters)
    - [7.9. PII parameters](../Guidelines.md#79-pii-parameters)
    - [7.10. Response formats](../Guidelines.md#710-response-formats)
        - [7.10.1. Clients-specified response format](../Guidelines.md#7101-clients-specified-response-format)
        - [7.10.2. Error condition responses](../Guidelines.md#7102-error-condition-responses)
    - [7.11. HTTP Status Codes](../Guidelines.md#711-http-status-codes)
    - [7.12. Client library optional](../Guidelines.md#712-client-library-optional)
- [8. CORS](../Guidelines.md#8-cors)
    - [8.1. Client guidance](../Guidelines.md#81-client-guidance)
        - [8.1.1. Avoiding preflight](../Guidelines.md#811-avoiding-preflight)
    - [8.2. Service guidance](../Guidelines.md#82-service-guidance)
- [9. Collections](../Guidelines.md#9-collections)
    - [9.1. Item keys](../Guidelines.md#91-item-keys)
    - [9.2. Serialization](../Guidelines.md#92-serialization)
    - [9.3. Collection URL patterns](../Guidelines.md#93-collection-url-patterns)
        - [9.3.1. Nested collections and properties](../Guidelines.md#931-nested-collections-and-properties)
    - [9.4. Big collections](../Guidelines.md#94-big-collections)
    - [9.5. Changing collections](../Guidelines.md#95-changing-collections)
    - [9.6. Sorting collections](../Guidelines.md#96-sorting-collections)
        - [9.6.1. Interpreting a sorting expression](../Guidelines.md#961-interpreting-a-sorting-expression)
    - [9.7. Filtering](../Guidelines.md#97-filtering)
        - [9.7.1. Filter operations](../Guidelines.md#971-filter-operations)
        - [9.7.2. Operator examples](../Guidelines.md#972-operator-examples)
        - [9.7.3. Operator precedence](../Guidelines.md#973-operator-precedence)
    - [9.8. Pagination](../Guidelines.md#98-pagination)
        - [9.8.1. Server-driven paging](../Guidelines.md#981-server-driven-paging)
        - [9.8.2. Client-driven paging](../Guidelines.md#982-client-driven-paging)
        - [9.8.3. Additional considerations](../Guidelines.md#983-additional-considerations)
    - [9.9. Compound collection operations](../Guidelines.md#99-compound-collection-operations)
- [**9a. Resource Design**](#9a-resource-design)
    - [**9a.1. Noun Resources**](#9a1-noun-resources)
    - [**9a.2. Verb Resources**](#9a2-verb-resources)
    - [**9a.3. Resource Modeling**](#9a3-resource-modeling)
- [10. Delta queries](#10-delta-queries)
    - [10.1. Delta links](../Guidelines.md#101-delta-links)
    - [10.2. Entity representation](../Guidelines.md#102-entity-representation)
    - [10.3. Obtaining a delta link](../Guidelines.md#103-obtaining-a-delta-link)
    - [10.4. Contents of a delta link response](../Guidelines.md#104-contents-of-a-delta-link-response)
    - [10.5. Using a delta link](../Guidelines.md#105-using-a-delta-link)
- [11. JSON standardizations](../Guidelines.md#11-json-standardizations)
    - [11.1. JSON formatting standardization for primitive types](../Guidelines.md#111-json-formatting-standardization-for-primitive-types)
    - [11.2. Guidelines for dates and times](../Guidelines.md#112-guidelines-for-dates-and-times)
        - [11.2.1. Producing dates](../Guidelines.md#1121-producing-dates)
        - [11.2.2. Consuming dates](../Guidelines.md#1122-consuming-dates)
        - [11.2.3. Compatibility](../Guidelines.md#1123-compatibility)
    - [11.3. JSON serialization of dates and times](../Guidelines.md#113-json-serialization-of-dates-and-times)
        - [11.3.1. The `DateLiteral` format](../Guidelines.md#1131-the-dateliteral-format)
        - [11.3.2. Commentary on date formatting](../Guidelines.md#1132-commentary-on-date-formatting)
    - [11.4. Durations](../Guidelines.md#114-durations)
    - [11.5. Intervals](../Guidelines.md#115-intervals)
    - [11.6. Repeating intervals](../Guidelines.md#116-repeating-intervals)
    - [**11.7. Evolvable Enums**](#117-evolvable-enums)
    - [**11.8. Dictionary Types**](#118-dictionary-types)
    - [**11.9. Ommitted Properties**](#119-ommitted-properties)
- [12. Versioning](../Guidelines.md#12-versioning)
    - [12.1. Versioning formats](../Guidelines.md#121-versioning-formats)
        - [12.1.1. Group versioning](../Guidelines.md#1211-group-versioning)
    - [12.2. When to version](../Guidelines.md#122-when-to-version)
    - [12.3. Definition of a breaking change](../Guidelines.md#123-definition-of-a-breaking-change)
- [**13. Long running operations**](#13-long-running-operations)
    - [**13.1. Resource based long running operations (RELO)**](../long-running-operations.md#131-resource-based-long-running-operations-relo)
    - [**13.2. Stepwise long running operations**](../long-running-operations.md#132-stepwise-long-running-operations)
        - [13.2.1. PUT](../Guidelines.md#1321-put)
        - [13.2.2. POST](../Guidelines.md#1322-post)
        - [13.2.3. POST, hybrid model](../Guidelines.md#1323-post-hybrid-model)
        - [13.2.4. Operations resource](../Guidelines.md#1324-operations-resource)
        - [13.2.5. Operation resource](../Guidelines.md#1325-operation-resource)
        - [13.2.6. Operation tombstones](../Guidelines.md#1326-operation-tombstones)
        - [13.2.7. The typical flow, polling](../Guidelines.md#1327-the-typical-flow-polling)
        - [13.2.8. The typical flow, push notifications](../Guidelines.md#1328-the-typical-flow-push-notifications)
        - [13.2.9. Retry-After](../Guidelines.md#1329-retry-after)
    - [13.3. Retention policy for operation results](../Guidelines.md#133-retention-policy-for-operation-results)
- [14. Throttling, Quotas, and Limits](../Guidelines.md#14-throttling-quotas-and-limits)
    - [14.1. Principles](../Guidelines.md#141-principles)
    - [14.2. Return Codes (429 vs 503)](../Guidelines.md#142-return-codes-429-vs-503)
    - [14.3. Retry-After and RateLimit Headers](../Guidelines.md#143-retry-after-and-ratelimit-headers)
    - [14.4. Service Guidance](../Guidelines.md#144-service-guidance)
        - [14.4.1. Responsiveness](../Guidelines.md#1441-responsiveness)
        - [14.4.2. Rate Limits and Quotas](../Guidelines.md#1442-rate-limits-and-quotas)
        - [14.4.3. Overloaded services](../Guidelines.md#1443-overloaded-services)
        - [14.4.4. Example Response](../Guidelines.md#1444-example-response)
    - [14.5. Caller Guidance](../Guidelines.md#145-caller-guidance)
    - [14.6. Handling callers that ignore Retry-After headers](../Guidelines.md#146-handling-callers-that-ignore-retry-after-headers)
- [**15. Push notifications via webhooks**](#15-push-notifications-via-webhooks)
    - [15.1. Scope](../Guidelines.md#151-scope)
    - [15.2. Principles](../Guidelines.md#152-principles)
    - [15.3. Types of subscriptions](../Guidelines.md#153-types-of-subscriptions)
    - [15.4. Call sequences](../Guidelines.md#154-call-sequences)
    - [15.5. Verifying subscriptions](../Guidelines.md#155-verifying-subscriptions)
    - [15.6. Receiving notifications](../Guidelines.md#156-receiving-notifications)
        - [15.6.1. Notification payload](../Guidelines.md#1561-notification-payload)
    - [15.7. Managing subscriptions programmatically](../Guidelines.md#157-managing-subscriptions-programmatically)
        - [15.7.1. Creating subscriptions](../Guidelines.md#1571-creating-subscriptions)
        - [15.7.2. Updating subscriptions](../Guidelines.md#1572-updating-subscriptions)
        - [15.7.3. Deleting subscriptions](../Guidelines.md#1573-deleting-subscriptions)
        - [15.7.4. Enumerating subscriptions](../Guidelines.md#1574-enumerating-subscriptions)
    - [15.8. Security](../Guidelines.md#158-security)
- [16. Unsupported requests](../Guidelines.md#16-unsupported-requests)
    - [16.1. Essential guidance](../Guidelines.md#161-essential-guidance)
    - [16.2. Feature allow list](../Guidelines.md#162-feature-allow-list)
        - [16.2.1. Error response](../Guidelines.md#1621-error-response)
- [17. Naming guidelines](../Guidelines.md#17-naming-guidelines)
    - [17.1. Approach](../Guidelines.md#171-approach)
    - [17.2. Casing](../Guidelines.md#172-casing)
    - [17.3. Names to avoid](../Guidelines.md#173-names-to-avoid)
    - [17.4. Forming compound names](../Guidelines.md#174-forming-compound-names)
    - [17.5. Identity properties](../Guidelines.md#175-identity-properties)
    - [17.6. Date and time properties](../Guidelines.md#176-date-and-time-properties)
    - [17.7. Name properties](../Guidelines.md#177-name-properties)
    - [17.8. Collections and counts](../Guidelines.md#178-collections-and-counts)
    - [17.9. Common property names](../Guidelines.md#179-common-property-names)
    - [**17.10. Type namespaces**](#1710-type-namespaces)
- [18. Appendix](../Guidelines.md#18-appendix)
    - [18.1. Sequence diagram notes](../Guidelines.md#181-sequence-diagram-notes)
        - [18.1.1. Push notifications, per user flow](../Guidelines.md#1811-push-notifications-per-user-flow)
        - [18.1.2. Push notifications, firehose flow](../Guidelines.md#1812-push-notifications-firehose-flow)
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

For scenarios where there is a need to persist a variable number of properties, a [dictionary type](./dictionary/index.md) may be useful.

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
