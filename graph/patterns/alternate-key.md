# Alternate Key Pattern

Microsoft Graph API Design Pattern

*The Alternate Key Pattern provides the ability to query for a single, specific entity identifiable through an alternative attribute (called key) that is not its unique identifier*

## Problem
--------

All entities in our system are identified by an UUID - which guarantees uniqueness. Often though, that same entity can also be uniquely identified by an alternative, more convenient attribute.

Take a look at the `user` entity: while the UUID remains a perfectly valid way to get the entity details, the `email` address is also an unique attribute that could be used to identify it.

While it is still possible to use the oData filter, such as

`serviceRoot/Users?$filter=email eq 'hello@microsoft.com'`, the returned result is wrapped in an array that needs to be unpacked.


## Solution
--------

oData offers entity addressing via an alternate key using the same parentheses-style convention as for the canonical key, with one difference: single-part alternate keys MUST specify the key property name to unambiguously determine the alternate key.

http://host/service/Employees(0) - Retrieves the employee with ID = 0
http://host/service/Employees(email='hello@microsoft.com') Retrieves the employee with the email matching `hello@microsoft.com`

## When to Use this Pattern
------------------------

This pattern works when the alternate key is good enough to identify a single entity that is properly namespaced; while it does not work if the resultset has more than one element.

In such case, we **strongly** advice to throw an exception and encourage the user to use `$filter` rather than returning the first result of the query 

## Example
-------

The same user identified via the alternate key SSN, the canonical (primary) key ID using the non-canonical long form with specified key property name, and the canonical short form without key property name

http://host/service/Employees/1a89ade6-9f59-4fea-a139-23f84e3aef66

http://host/service/Employees(ssn='123-45-6789')

http://host/service/Employees(email='hello@microsoft.com')





