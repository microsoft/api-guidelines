# Alternate Key Pattern

Microsoft Graph API Design Pattern

_The Alternate Key Pattern provides the ability to query for a single, specific resource identifiable through an alternative property (called key) that is not its unique identifier_

## Problem

---

The resources exposed in Graph are identified by an [UUID (Universally Unique Identifier)](https://en.wikipedia.org/wiki/Universally_unique_identifier) - which guarantees uniqueness inside the same resource type. Often though, that same resource can also be uniquely identified by an alternative, more convenient property that provides a better developer experience.

Take a look at the `user` resource: while the `id` remains a perfectly valid way to get the resource details, the `mail` address is also an unique property that could be used to identify it.

While it is still possible to use the oData filter, such as

`https://graph.microsoft.com/v1.0/users?$filter=mail eq 'bob@contoso.com'`, the returned result is wrapped in an array that needs to be unpacked.

## Solution

---

oData offers resource addressing via an alternate key using the same parentheses-style convention as for the canonical key, with one difference: single-part alternate keys MUST specify the key property name to unambiguously determine the alternate key.

https://graph.microsoft.com/v1.0/users(0) - Retrieves the employee with ID = 0
https://graph.microsoft.com/v1.0/users(email='bob@contoso.com') Retrieves the employee with the email matching `bob@contoso.com`

## When to Use this Pattern

---

This pattern works and makes sense when the alternate key is good enough to identify a single resource and provides an useful alternative to the client; while it does not work if the resultset has more than one element.

In such case, the system SHOULD throw an exception and encourage the user to use `$filter` rather than returning the first result of the query

## Example

---

The same user identified via the alternate key SSN, the canonical (primary) key ID using the non-canonical long form with specified key property name, and the canonical short form without key property name

https://graph.microsoft.com/v1.0/users/1a89ade6-9f59-4fea-a139-23f84e3aef66

https://graph.microsoft.com/v1.0/users(ssn='123-45-6789')

https://graph.microsoft.com/v1.0/users(email='bob@contoso.com')

All of the 3 will yield the sare response:


```json
{
   "givenName": "Bob",
   "jobTitle": "Retail Manager",
   "mail": "bob@contoso.com",
   "mobilePhone": "+1 425 555 0109",
   "officeLocation": "18/2111",
   "preferredLanguage": "en-US",
   "surname": "Vance",
   "userPrincipalName": "bob@contoso.com",
   "id": "1a89ade6-9f59-4fea-a139-23f84e3aef66"
}
```

