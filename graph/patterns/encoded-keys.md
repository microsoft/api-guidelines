# Encoded Key Properties

Microsoft Graph API Design Pattern

Key properties should be encoded with [base64url encoding](https://datatracker.ietf.org/doc/html/rfc4648#section-5) when their values can include the `/` character. 


## Problem

OData URLs that identify an individual entity within a collection will contain the key for that entity.
These keys can sometimes include reserved URL characters, and the [OData standard](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part2-url-conventions.html#sec_URLParsing) indicates that these characters should be percent-encoded. 
Due to a long-standing bug in Microsoft Graph, and one that cannot be fixed in order to maintain backwards compatibility, percent-encoding the `/` character is not possible for Microsoft Graph APIs.
This means that key properties cannot contain the `/` character, which can be limiting to the API design.

## Solution

A key property that may contain the `/` character should be base64url encoded, and each instance of the property should be encoded regardless if that particular value contains the `/` character.
The entity may additionally include a non-key property that contains the plaintext of the key property.
This property should have the same name as the key property with `Plaintext` appended. 
The collection may also support filtering on the plaintext property.

## When to use this pattern

This pattern should be used whenever the value of a key property may contain a `/` character.

## Issues and considerations

Encoding a key property often obfuscates the natural way that a client identifies an entity, particularly entities that are related to extenal services or standards.
A plaintext property that allows filtering can help to reduce this impact, but filtering itself has semantic limitations in OData (for example, if the client wants to navigate to a navigation property of the entity, using a filter does not allow this).

## Example

```xml
<EntityType Name="foo">
  <Key>
    <PropertyRef Name="id" />
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false">
    <Annotation Term="Org.OData.Core.V1.Description" String="The base64url encoding of the idPlaintext property" />
  </Property>
  <Property Name="idPlaintext" Type="Edm.String" Nullable="false" />
  ...
</EntityType>
```
```json
GET /foos/dGhpcyBpcyBhbiBpZCB3aXRoIC8

200 OK
{
  "id": "dGhpcyBpcyBhbiBpZCB3aXRoIC8",
  "idPlaintext": "this is an id with /",
  ...
}
```
```json
GET /foos('dGhpcyBpcyBhbiBpZCB3aXRoIC8')

200 OK
{
  "id": "dGhpcyBpcyBhbiBpZCB3aXRoIC8",
  "idPlaintext": "this is an id with /",
  ...
}
```
```json
GET /foos?$filter=idPlaintext eq 'this is an id with /'

200 OK
{
  "value": [
    {
      "id": "dGhpcyBpcyBhbiBpZCB3aXRoIC8",
      "idPlaintext": "this is an id with /",
      ...
    }
  ]
}
```
