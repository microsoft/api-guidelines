# Encoded Key Properties

Microsoft Graph API Design Pattern

Key properties should be encoded with [base64url encoding](https://datatracker.ietf.org/doc/html/rfc4648#section-5) when their values can include the `/` character. 


## Problem

OData URLs that identify an individual entity within a collection will contain the key for that entity.
These keys can sometimes include reserved URL characters, and the [OData standard](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part2-url-conventions.html#sec_URLParsing) indicates that these characters should be percent-encoded. 
Due to a long-standing bug in Microsoft Graph, and one that cannot be fixed in order to maintain backwards compatibility, percent-encoding the `/` character is not possible for Microsoft Graph APIs.

## Solution

*Describe how to implement the solution to solve the problem.*

*Describe related patterns.*

## When to use this pattern

*Describe when and why the solution is applicable and when it might not be.*

## Issues and considerations

*Describe tradeoffs of the solution.*

## Example

*Provide a short example from real life.*
