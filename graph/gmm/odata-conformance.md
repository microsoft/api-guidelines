---
title: OData conformance
---

# OData Conformance Requirements for Microsoft Graph

The conformance requirements are listed below in different levels. These levels correspond to whether a service must implement them or if they are optional. The actual OData conformance requirements and details (sections mentioned beside the point) can be found at this [**link.**](http://docs.oasis-open.org/odata/odata/v4.0/errata03/os/complete/part1-protocol/odata-v4.0-errata03-os-part1-protocol-complete.html#_Toc453752324)

## Level 1

Minimum Requirements a workload service needs to meet in order to onboard onto graph successfully.

- MUST return data according to at least one of the OData defined formats (section 7). We require JSON
- MUST support server-driven paging when returning partial results (section 11.2.5.7)
- MUST successfully parse the request according to [OData-ABNF][2] for any supported system query string options.
- MUST expose only data types defined in [OData-CSDL][2]
- MUST NOT violate any OData update semantics (section 11.4 and all subsections)
- MUST publish metadata at $metadata according to [OData-CSDL][2] (section 11.1.2)
- MUST support the resource path conventions defined in [OData URL][2]

## Level 2

These requirements are highly recommended, but are still optional. This may vary based on the workload service, where some conformance requirements must be implemented.

- Return the appropriate OData-Version header (section 8.1.5)
- Conform to the semantics the following headers, or fail the request
  - Accept (section 8.2.1)
  - OData-MaxVersion (section 8.2.7)
- Include edit links (explicitly or implicitly) for all updatable or deletable resources according to [OData-Atom][2] and [OData-JSON][2]
- Support POST of new entities to insertable entity sets (section 11.4.1.5 and 11.4.2.1)
- Support PATCH to all edit URLs for updatable resources (section 11.4.3)
- Support DELETE to all edit URLs for deletable resources (section 11.4.5)
- Support DELETE to $ref to remove an entity from an updatable navigation property (section 11.4.6.2)
- Return a Location header with the edit URL or read URL of a created resource (section 11.4.1.5)
- Support $select (section11.2.4.1)
- Support casting to a derived type according to [OData URL][2] if derived types are present in the model
- Support $top (section 11.2.5.3)
- Support $filter (section 11.2.5.1)
- Support eq, ne filter operations on properties of entities in the requested entity set (section 11.2.5.1.1)
- Support the $skip system query option (section 11.2.5.4)
- Support the $count system query option (section 11.2.5.5)
- Support $orderby asc and desc on individual properties (section 11.2.5.2)

## Level 3

These Odata conformance requirements can be implemented based on the discretion/requirements of the workload. To decide whether a workload service needs to implement one or more these, please contact Microsoft Graph team.

- Support $expand (section 11.2.4.2)
- Support POST of new related entities to updatable navigation properties (section 11.4.6.1)
- Support POST to $ref to add an existing entity to an updatable related collection (section 11.4.6.1)
- Support PUT to $ref to set an existing single updatable related entity (section 11.4.6.3)
- Support if-match header in update/delete of any resources returned with an ETag (section 11.4.1.1)
- Include the OData-EntityId header in response to any create or upsert operation that returns 204 No Content (Section 8.3.3)
- Support Upserts (section 11.4.4)
- Support PUT and PATCH to an individual primitive (section 11.4.9.1) or complex (section 11.4.9.3) property (respectively)
- Support DELETE to set an individual property to null (section 11.4.9.2)
- Support deep inserts (section 11.4.2.2)
- Support /$value on media entities (section 4.10. in [OData URL][2] and individual properties (section 11.2.3.1)
- Support aliases in $filter expressions (section 11.2.5.1.3)
- Support additional filter operations (section 11.2.5.1.1) and MUST return 501 Not Implemented for any unsupported filter operations (section 9.3.1)
- Support the canonical functions (section 11.2.5.1.2) and MUST return 501 Not Implemented for any unsupported canonical functions (section 9.3.1)
- Support $filter on expanded entities (section 11.2.4.2.1)
- Support the $search system query option (section 11.2.5.6)
- Support $expand (section 11.2.4.2)
- Support the lambda operators any and all on navigation- and collection-valued properties (section 5.1.1.5 in [OData URL][2])
- Support $expand (section 11.2.4.2)
  - Support returning references for expanded properties (section 11.2.4.2)
  - Support $filter on expanded entities (section 11.2.4.2.1)
  - Support cast segment in expand with derived types (section 11.2.4.2.1)
  - Support $orderby asc and desc on individual properties (section 11.2.4.2.1)
  - Support the $count system query option for expanded properties (section 11.2.4.2.1)
  - Support $top and $skip on expanded properties (section 11.2.4.2.1)
  - Support $search on expanded properties (section 11.2.4.2.1)
  - Support $levels for recursive expand (section 11.2.4.2.1.1)
- Support batch requests (section11.7 and all subsections)
- Support Asynchronous operations (section 8.2.8.8)
- Support Delta change tracking (section 8.2.8.6)
- Support cross-join queries defined in [OData URL][2]
- Support a conforming OData service interface over metadata (section 11.1.3)

[2]: http://docs.oasis-open.org/odata/odata/v4.0/errata03/os/complete/part1-protocol/odata-v4.0-errata03-os-part1-protocol-complete.html#ABNF
