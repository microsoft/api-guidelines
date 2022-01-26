# Navigation Property

Microsoft Graph API Design Pattern

*A navigation property is used to identify a relationship between two resources.*

## Problem
--------

Resources often contain information that identifies other related resources. Often that information is contained in a returned representation as an id value. In order for a client to access the related resource it must request the primary resource, read the id value of the related resource and then construct a URL to the related resource using the Id value. This requires at least two round trips and requires the client know how to construct the URL to the related resource.

## Solution
--------

Navigation properties are an OData convention that allows an API designer to create a special property in a model that references an related entity. In the HTTP API this property name translates to a path segment that can be appended to the URL of the primary resource in order to access a representation of the related resource. This prevents the client from needing to know how to construct the URL to the related resource and the client does not need to retrieve the primary resource if it is only interested in the related resource.  It is the responsibility of the API implementation to determine the ID of the related resource and return the representation of the related entity.

Additionally, using the OData Expand query parameter, related entities can be transcluded into the primary entity so both can be retrieved in a single round trip.

## Issues and Considerations
-------------------------

Cross workload expands don't work.
 

## When to Use this Pattern
------------------------

*Describe when and why the solution is applicable and when it may not.*

 

## Example
-------

*Provide a short example from real life*

 

 

 
