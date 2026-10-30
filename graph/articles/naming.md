# Naming

## 1. Approach

Naming policies should aid developers in discovering functionality without having to constantly refer to documentation.
Use of common patterns and standard conventions greatly aids developers in correctly guessing common property names and meanings.
Services SHOULD use verbose naming patterns and MUST NOT use abbreviations other than acronyms that are the dominant mode of expression in the domain being represented by the API, (e.g. Url).

## 2. Casing

- Acronyms SHOULD follow the casing conventions as though they were regular words (e.g. Url).
- All identifiers including namespaces, entityTypes, entitySets, properties, actions, functions and enumeration values MUST use lowerCamelCase.
- HTTP headers are the exception and SHOULD use standard HTTP convention of Capitalized-Hyphenated-Terms.

## 3. Names to avoid

Certain names are so overloaded in API domains that they lose all meaning or clash with other common usages in domains that cannot be avoided when using REST APIs, such as OAUTH.
Services SHOULD NOT use the following names:

- Context
- Scope
- Resource

## 4. Forming compound names

- Services SHOULD avoid using articles such as 'a', 'the', 'of' unless needed to convey meaning.
  - e.g. names such as aUser, theAccount, countOfBooks SHOULD NOT be used, rather user, account, bookCount SHOULD be preferred.
- Services SHOULD add a type to a property name when not doing so would cause ambiguity about how the data is represented or would cause the service not to use a common property name.
- When adding a type to a property name, services MUST add the type at the end, e.g. createdDateTime.

## 5. Identity properties

- Services MUST use string types for identity properties.
- For OData services, the service MUST use the OData @id property to represent the canonical identifier of the resource.
- Services MAY use the simple 'id' property to represent a local or legacy primary key value for a resource.
- Services SHOULD use the name of the relationship postfixed with 'Id' to represent a foreign key to another resource, e.g. subscriptionId.
  - The content of this property SHOULD be the canonical ID of the referenced resource.

## 6. Date and time properties

- For properties requiring both date and time, services MUST use the suffix 'DateTime'.
- For properties requiring only date information without specifying time, services MUST use the suffix 'Date', e.g. birthDate.
- For properties requiring only time information without specifying date, services MUST use the suffix 'Time', e.g. appointmentStartTime.

## 7. Name properties

- For the overall name of a resource typically shown to users, services MUST use the property name 'displayName'.
- Services MAY use other common naming properties, e.g. givenName, surname, signInName.

## 8. Collections and counts

- Services MUST name collections as plural nouns or plural noun phrases using correct English.
- Services MAY use simplified English for nouns that have plurals not in common verbal usage.
  - e.g. schemas MAY be used instead of schemata.
- Services MUST name counts of resources with a noun or noun phrase suffixed with 'Count'.

## 9. Common property names

Where services have a property, whose data matches the names below, the service MUST use the name from this table.
This table will grow as services add terms that will be more commonly used.
Service owners adding such terms SHOULD propose additions to this document.

|                     |   |
|-------------------- | - |
 attendees            |
 body                 |
 completedDateTime    | **NOTE** completionDateTime may be used for cases where the timestamp represents a point in the future |
 createdDateTime      |
 childCount           |
 children             |
 contentUrl           |
 country              |
 createdBy            |
 displayName          |
 errorUrl             |
 eTag                 |
 event                |
 expirationDateTime   |
 givenName            |
 jobTitle             |
 kind                 |
 id                   |
 lastModifiedDateTime |
 location             |
 memberOf             |
 message              |
 name                 |
 owner                |
 people               |
 person               |
 postalCode           |
 photo                |
 preferredLanguage    |
 properties           |
 signInName           |
 surname              |
 tags                 |
 userPrincipalName    |
 webUrl               |
