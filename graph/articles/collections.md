# Collections

## 1. Item keys

Services SHOULD support durable identifiers for each item in the collection, and that identifier SHOULD be represented in JSON as "id". These durable identifiers are often used as item keys.

Collections MAY support delta queries, see the [Change Tracking pattern](../patterns/change-tracking.md) section for more details.

## 2. Serialization

Collections are represented in JSON using standard array notation for `value` property.

## 3. Collection URL patterns

While there are multiple collections located directly under the Graph root going forward, you MUST have a singleton for the top-level segment and scope collections to an appropriate singleton. Collection names SHOULD be plural nouns when possible. Collection names shouldn't use suffixes, such as "Collection" or "List".

For example:

```http
GET https://graph.microsoft.com/v1.0/teamwork/devices
```

Collections elements MUST be addressable by a unique id property. The id property MUST be a String and MUST be unique within the collection. The id property MUST be represented in JSON as "id".
For example:

```http
GET https://graph.microsoft.com/beta/teamwork/devices/0f3ce432-e432-0f3c-32e4-3c0f32e43c0f
```

Where:

- "https://graph.microsoft.com/beta/teamwork" - the service root represented as the combination of host (site URL) + the root path to the service.
- "devices" – the name of the collection, unabbreviated, pluralized.
- "0f3ce432-e432-0f3c-32e4-3c0f32e43c0f" – the value of the unique id property that MUST be the raw string/number/guid value with no quoting but properly escaped to fit in a URL segment.

### 3.1. Nested collections and properties

Collection items MAY contain other collections.
For example, a devices collection MAY contain device resources that have multiple mac addresses:

```http
GET https://graph.microsoft.com/beta/teamwork/devices/0f3ce432-e432-0f3c-32e4-3c0f32e43c0f
```

```json

{
  "value": {
    "@odata.type": "#microsoft.graph.teamworkDevice",
    "id": "0f3ce432-e432-0f3c-32e4-3c0f32e43c0f",
    "deviceType": "CollaborationBar",
    "hardwareDetail": {
      "serialNumber": "0189",
      "uniqueId": "5abcdefgh",
      "macAddresses": [],
      "manufacturer": "yealink",
      "model": "vc210"
    },
    ...    
  }
}
```

## 4. Big collections

As data grows, so do collections.
Services SHOULD support server-side pagination from day one even for all collections, as adding pagination is a breaking change.
When multiple pages are available, the serialization payload MUST contain the opaque URL for the next page as appropriate.
Refer to the [paging guidance](../Guidelines-deprecated.md#98-pagination) for more details.

Clients MUST be resilient to collection data being either paged or nonpaged for any given request.

```json
{
  "value":[
    { "id": "Item 1","price": 9 95,"sizes": null},
    { … },
    { … },
    { "id": "Item 99","price": 5 99,"sizes": null}
  ],
  "@nextLink": "{opaqueUrl}"
}
```

## 5. Changing collections

POST requests are not idempotent.
This means that two POST requests sent to a collection resource with exactly the same payload MAY lead to multiple items being created in that collection.
This is often the case for insert operations on items with a server-side generated id.
For additional information refer to [Upsert pattern](../patterns/upsert.md).

For example, the following request:

```http
POST https://graph.microsoft.com/beta/teamwork/devices
```

Would lead to a response indicating the location of the new collection item:

```http
201 Created
Location: https://graph.microsoft.com/beta/teamwork/devices/123
```

And once executed again, would likely lead to another resource:

```http
201 Created
Location: https://graph.microsoft.com/beta/teamwork/devices/124
```

## 6. Sorting collections

The results of a collection query MAY be sorted based on property values.
The property is determined by the value of the _$orderBy_ query parameter.

The value of the _$orderBy_ parameter contains a comma-separated list of expressions used to sort the items.
A special case of such an expression is a property path terminating on a primitive property.

The expression MAY include the suffix "asc" for ascending or "desc" for descending, separated from the property name by one or more spaces.
If "asc" or "desc" is not specified, the service MUST order by the specified property in ascending order.

NULL values MUST sort as "less than" non-NULL values.

Items MUST be sorted by the result values of the first expression, and then items with the same value for the first expression are sorted by the result value of the second expression, and so on.
The sort order is the inherent order for the type of the property.

For example:

```http
GET https://graph.microsoft.com/beta/teamwork/devices?$orderBy=companyAssetTag
```

Will return all devices sorted by companyAssetTag in ascending order.

For example:

```http
GET https://graph.microsoft.com/beta/teamwork/devices?$orderBy=companyAssetTag desc
```

Will return all devices sorted by companyAssetTag in descending order.

Sub-sorts can be specified by a comma-separated list of property names with OPTIONAL direction qualifier.

For example:

```http
GET https://graph.microsoft.com/beta/teamwork/devices?$orderBy=companyAssetTag desc,activityState
```

Will return all devices sorted by companyAssetTag in descending order and a secondary sort order of activityState in ascending order.

Sorting MUST compose with filtering see [Odata 4.01 spec](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part2-url-conventions.html#_Toc31361038) for more details.

### 6.1. Interpreting a sorting expression

Sorting parameters MUST be consistent across pages, as both client and server-side paging is fully compatible with sorting.

If a service does not support sorting by a property named in a _$orderBy_ expression, the service MUST respond with an error message as defined in the Responding to Unsupported Requests section.

## 7. Filtering

The _$filter_ querystring parameter allows clients to filter a collection of resources that are addressed by a request URL.
The expression specified with _$filter_ is evaluated for each resource in the collection, and only items where the expression evaluates to true are included in the response.
Resources for which the expression evaluates to false or to null, or which reference properties that are unavailable due to permissions, are omitted from the response.

Example: return all devices with activity state equal to 'Active'

```http
GET https://graph.microsoft.com/beta/teamwork/devices?$filter=(activityState eq 'Active') 
```

The value of the _$filter_ option is a Boolean expression.

### 7.1. Filter operations

Services that support _$filter_ SHOULD support the following minimal set of operations.

Operator             | Description           | Example
-------------------- | --------------------- | -----------------------------------------------------
Comparison Operators |                       |
eq                   | Equal                 | city eq 'Redmond'
ne                   | Not equal             | city ne 'London'
gt                   | Greater than          | price gt 20
ge                   | Greater than or equal | price ge 10
lt                   | Less than             | price lt 20
le                   | Less than or equal    | price le 100
Logical Operators    |                       |
and                  | Logical and           | price le 200 and price gt 3.5
or                   | Logical or            | price le 3.5 or price gt 200
not                  | Logical negation      | not price le 3.5
Grouping Operators   |                       |
( )                  | Precedence grouping   | (priority eq 1 or city eq 'Redmond') and price gt 100

Services MUST use the following operator precedence for supported operators when evaluating _$filter_ expressions.
Operators are listed by category in order of precedence from highest to lowest.
Operators in the same category have equal precedence:

| Group           | Operator | Description           |
|:----------------|:---------|:----------------------|
| Grouping        | ( )      | Precedence grouping   |
| Unary           | not      | Logical Negation      |
| Relational      | gt       | Greater Than          |
|                 | ge       | Greater Than or Equal |
|                 | lt       | Less Than             |
|                 | le       | Less Than or Equal    |
| Equality        | eq       | Equal                 |
|                 | ne       | Not Equal             |
| Conditional AND | and      | Logical And           |
| Conditional OR  | or       | Logical Or            |

## 8. Pagination

RESTful APIs that return collections MAY return partial sets.
Consumers of these services MUST expect partial result sets and correctly page through to retrieve an entire set.

There are two forms of pagination that MAY be supported by RESTful APIs.
Server-driven paging allows servers to even out load across clients and mitigates against denial-of-service attacks by forcibly paginating a request over multiple response payloads.
Client-driven paging enables clients to request only the number of resources that it can use at a given time.

Sorting and Filtering parameters MUST be consistent across pages, because both client- and server-side paging is fully compatible with both filtering and sorting.

### 8.1. Server-driven paging

Paginated responses MUST indicate a partial result by including a `@odata.nextLink` token in the response.
The absence of a `nextLink` token means that no additional pages are available, see [Odata 4.01 spec](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html#sec_ServerDrivenPaging) for more details.

Clients MUST treat the `nextLink` URL as opaque, which means that query options may not be changed while iterating over a set of partial results.

Example:

```http
GET https://graph.microsoft.com/beta/teamwork/devices
Accept: application/json

HTTP/1.1 200 OK
Content-Type: application/json

{
  "value": [...],
  "@odata.nextLink": "{opaqueUrl}"
}
```

### 8.2. Client-driven paging

Clients MAY use _$top_ and _$skip_ query parameters to specify a number of results to return and an offset into the collection.

The server SHOULD honor the values specified by the client; however, clients MUST be prepared to handle responses that contain a different page size or contain a `@odata.nextLink` token.

When both _$top_ and _$skip_ are given by a client, the server SHOULD first apply _$skip_ and then _$top_ on the collection.

Note: If the server can't honor _$top_ and/or _$skip_, the server MUST return an error to the client informing about it instead of just ignoring the query options.
This will avoid the risk of the client making assumptions about the data returned.

Example:

```http
GET https://graph.microsoft.com/beta/teamwork/devices?$top=5&$skip=2 

Accept: application/json

HTTP/1.1 200 OK
Content-Type: application/json

{
   "value": [...]
}
```

### 8.3. Additional considerations

**Stable order prerequisite:** Both forms of paging depend on the collection of items having a stable order.
The server MUST supplement any specified order criteria with additional sorts (typically by key) to ensure that items are always ordered consistently.

**Missing/repeated results:** Even if the server enforces a consistent sort order, results MAY be missing or repeated based on creation or deletion of other resources.
Clients MUST be prepared to deal with these discrepancies.
The server SHOULD always encode the record ID of the last read record, helping the client in the process of managing repeated/missing results.

**Combining client- and server-driven paging:** Note that client-driven paging does not preclude server-driven paging.
If the page size requested by the client is larger than the default page size supported by the server, the expected response would be the number of results specified by the client, paginated as specified by the server paging settings.

**Page Size:** Clients MAY request server-driven paging with a specific page size by specifying a _$maxpagesize_ preference.
The server SHOULD honor this preference if the specified page size is smaller than the server's default page size.

**Paginating embedded collections:** It is possible for both client-driven paging and server-driven paging to be applied to embedded collections.
If a server paginates an embedded collection, it MUST include additional `nextLink` tokens as appropriate.

**Recordset count:** Developers who want to know the full number of records across all pages, MAY include the query parameter _$count=true_ to tell the server to include the count of items in the response.

## 9. Compound collection operations

Filtering, Sorting and Pagination operations MAY all be performed against a given collection.
When these operations are performed together, the evaluation order MUST be:

1. **Filtering**. This includes all range expressions performed as an AND operation.
2. **Sorting**. The potentially filtered list is sorted according to the sort criteria.
3. **Pagination**. The materialized paginated view is presented over the filtered, sorted list. This applies to both server-driven pagination and client-driven pagination.

## 10. Empty Results

When a filter is performed on a collection and the result set is empty you MUST respond with a valid response body and a 200 response code.
In this example the filters supplied by the client resulted in a empty result set.
The response body is returned as normal and the _value_ attribute is set to a empty collection.
You SHOULD maintain consistency in your API whenever possible.

```http
GET https://graph.microsoft.com/beta/teamwork/devices?$filter=('deviceType'  eq 'Collab' or companyAssetTa eq 'Tag1')
Accept: application/json

HTTP/1.1 200 OK
Content-Type: application/json

{
   "value": []
}
```
