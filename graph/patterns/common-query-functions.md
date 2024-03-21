# Exposing functions for common queries

Microsoft Graph API Design Pattern

*Exposing functions for common queries is providing functions with friendly names as an additional mechanism for clients to make certain kinds of requests that involve the use of OData query options.*

## Problem

Some APIs are very data-focused.
These APIs are generally capable of providing robust `$fitler`ing and `$expand`ing functionality.
However, the OData query options that need to be used by clients can become complicated and confusing.
Because of this, API producers have a tendency to avoid supporting query options that require client developers to have OData experience; avoiding these query options results in APIs that are not fully featured and are not externally extensible, limiting what clients can actually accomplish via the APIs.

## Solution

The solution to this problem is to provide support for the query options and to additionally define OData functions that act as aliases for a specific set of query options.
Doing this gives client developers a low barrier-of-entry to using the API (familiarizing themselves with the shape of the data), and it also gives those developers extensiblity in the future to grow beyond the basic cases to write clients that are specific to their own business needs.
The functions can further provide discoverability by using names for concepts that customers are familiar with, letting them know that those concepts are supported.

## When to use this pattern

This pattern can be employed in almost any circumstance.
It is also able to be used once an API as shipped and in any order.
It is ok to ship a data-focused API with robust `$filter`ing support, and *later* ship functions that act in the same way as certain specific filters.
It is likewise fine to ship a handful of functions and then later ship an API that is a collection of data with a more fully-featured set of filters.

## Issues and considerations

The tradeoff with this pattern is ensuring that the functions don't become so numerous that they remove the aliasing benefit.
It is important to remember that the functions exist to increase discoverability, decrease onboarding costs, and prevent client mistakes writing complicated OData queries.
If a function is being considered that does not directly address one of these issues, it likely shouldn't be introduced.

## Example

Suppose that there is a collection of attempted sign-ins for an organization. The model might look like this:

```xml
<EntityContainer Name="container">
  <EntitySet Name="signInAttempts" EntityType="self.signInAttempt" />
</EntityContainer>

<EntityType Name="signInAttempt">
  <Key>
    <PropertyRef Name="id" />
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false" />

  <Property Name="firstFactorUsed" Type="self.authenticationFactor" Nullable="false" />
  <Property Name="otherFactorsUsed" Type="Collection(self.authenticationFactor)" Nullable="false" />
  <Property Name="isSuccessful" Type="Edm.Boolean" Nullable="false" />
</EntityType>

<ComplexType Name="authenticationFactor">
  <!--other properties here-->
</ComplexType>
```

Clients could then request all of the successful, multifactor sign-in attempts by calling:

```http
GET /signInAttempts?$filter=isSuccessful eq true and otherFactorsUsed/$count ne 0

HTTP/1.1 200 OK
{
  "value": [
    {
      "id": "{id1}",
      "firstFactorUsed": {
        // other properties here
      },
      "otherFactorsUsed": [
        {
          // other properties here
        },
        ...
      ],
      "isSuccessful": true
    },
    ...
  ]
}
```

An API producer could alias this common filtering use case by adding a function to the CSDL:

```xml
<Function Name="successfulMultifactorSignIns" IsBound="true">
  <Parameter Name="bindingParameter" Type="Collection(self.signInAttempt)" Nullable="false" />
  <ReturnType Type="self.signInAttempt" />
</Function>
```

Clients would then be able to call:

```http
GET /signInAttempts/successfulMultifactorSignIns

HTTP/1.1 200 OK
{
  "value": [
    {
      "id": "{id1}",
      "firstFactorUsed": {
        // other properties here
      },
      "otherFactorsUsed": [
        {
          // other properties here
        },
        ...
      ],
      "isSuccessful": true
    },
    ...
  ]
}
```

to retreive the same data.
