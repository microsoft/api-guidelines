# Query option aliasing

Microsoft Graph API Design Pattern

*Query option aliasing is providing functions with friendly names as an additional mechanism for clients to make certain kinds of requests that involve the use of OData query options.*


## Problem

Some APIs are very data-focused.
These APIs are generally capable of providing robust `$fitler`ing and `$expand`ing functionality.
However, the OData query options that need to be used by clients can become complicated and confusing.
Because of this, API producers have a tendency to avoid supporting query options that require client developers to have OData experience; avoiding these query options results in APIs that are not fully featured and are not externally extensible, limiting what clients can actually accomplish via the APIs.

## Solution

*Describe how to implement the solution to solve the problem.*

*Describe related patterns.*

## When to use this pattern

*Describe when and why the solution is applicable and when it might not be.*

## Issues and considerations

*Describe tradeoffs of the solution.*

## Example

*Provide a short example from real life.*
