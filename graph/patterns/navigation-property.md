# Navigation Property

Microsoft Graph API Design Pattern

*A navigation property is used to identify a relationship between two resources.*

## Problem
--------

Resources often contain information that identifies other related resources. Usually that information is contained in a returned representation as an id value. In order for a client to access the related resource it must request the primary resource, read the id value of the related resource and then construct a URL to the related resource using the Id value. This requires at least two round trips and requires the client know how to construct the URL to the related resource.

## Solution
--------

Navigation properties are an OData convention that allows an API designer to create a special kind of property in a model that references an related entity. In the HTTP API this property name translates to a path segment that can be appended to the URL of the primary resource in order to access a representation of the related resource. This prevents the client from needing to know any additional information on how to construct the URL to the related resource and the client does not need to retrieve the primary resource if it is only interested in the related resource.  It is the responsibility of the API implementation to determine the Id of the related resource and return the representation of the related entity.

Additionally, using the OData Expand query parameter, related entities can be transcluded into the primary entity so both can be retrieved in a single round trip.

## Issues and Considerations
-------------------------

In the current Microsoft Graph implementation, support for navigation properties is limited to entities within the same backend service or the user entity.  
 
Implementing support for accessing the "$ref" of a navigation property allows a caller to return just the URL of related resource. e.g. `/user/23/manager/$ref`. This is useful when a client wishes to identity the related resource but doesn't need all of its properties.

## When to Use this Pattern
------------------------

The use of navigation properties is preferred over 

 

## Example
-------

*Provide a short example from real life*

 

 

 
