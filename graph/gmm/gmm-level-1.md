---
title: GMM level 1
owner: mastaffo
---

# GMM level 1

## Naming

| Id                      | Name                                                                                           | Severity |
| ----------------------- | ---------------------------------------------------------------------------------------------- | -------- |
| <a id="N0001">N0001</a> | ✔ DO use `lowerCamelCase` for _all_ names.                                                     | Error    |
| <a id="N0002">N0002</a> | ✔ DO use singular nouns for type names and properties with cardinality `1`.                    | Error    |
| <a id="N0003">N0003</a> | ✔ DO use plural nouns for collections and properties with cardinality `*`.                     | Error    |
| <a id="N0004">N0004</a> | ✖ AVOID using brand names in type or property names.                                           | Error    |
| <a id="N0005">N0005</a> | ✖ AVOID using acronyms or abbreviations unless the abbreviation is extremely well known.       | Error    |
| <a id="N0006">N0006</a> | ✔ DO case two-letter acronyms with the same case.                                              | Error    |
| <a id="N0007">N0007</a> | ✔ DO case three+ letter acronyms the same as a normal word.                                    | Error    |
| <a id="N0008">N0008</a> | ✖ DO NOT suffix property names with primitive type information unless the type is temporal.    | Error    |
| <a id="N0009">N0009</a> | ✔ DO suffix `Edm.Date` property names with `…Date`.                                            | Error    |
| <a id="N0010">N0010</a> | ✔ DO suffix `Edm.Time` property names with `…Time`.                                            | Error    |
| <a id="N0011">N0011</a> | ✔ DO suffix `Edm.DateTime` property names with `…DateTime`.                                    | Error    |
| <a id="N0012">N0012</a> | ✖ DO NOT prefix properties with the name of the type unless it is significantly more readable. | Error    |
| <a id="N0013">N0013</a> | ✔ CONSIDER using a property name from the shared property name list.                           | Info     |
| <a id="N0014">N0014</a> | ✖ AVOID using `type` as the name of a property.                                                | Warning  |
| <a id="N0015">N0015</a> | ✖ AVOID using `Mail` in property names. Instead use `Email`.                                   | Warning  |

## Modeling

| Id                      | Name                                                                                            | Severity |
| ----------------------- | ----------------------------------------------------------------------------------------------- | -------- |
| <a id="M0001">M0001</a> | ✔ DO make the key of every entity a single property with name `id` and type `Edm.String`.       | Error    |
| <a id="M0002">M0002</a> | ✔ DO make every entity inherit from `microsoft.graph.entity`.                                   | Error    |
| <a id="M0003">M0003</a> | ✖ DO NOT use parallel collections; use collections of complex types instead.                    | Error    |
| <a id="M0004">M0004</a> | ✔ CONSIDER use a proper collection rather than `property1`, `property2`, etc.                   | Warning  |
| <a id="M0005">M0005</a> | ✖ AVOID denormalization unless it is necessary for developer scenarios.                         | Warning  |
| <a id="M0006">M0006</a> | ✔ CONSIDER using an evolvable enumeration for enumerations that will add members in the future. | Warning  |
| <a id="M0007">M0007</a> | ✖ AVOID operations such as actions and functions whenever possible.                             | Warning  |
| <a id="M0008">M0008</a> | ✔ CONSIDER using inheritance when types share three or more properties in common.               | Warning  |
| <a id="M0009">M0009</a> | ✖ DO NOT use a complex type with an `id`. Instead use an entity.                                | Error    |
| <a id="M0010">M0010</a> | ✔ CONSIDER making enums with two options a boolean.                                             | Warning  |
| <a id="M0011">M0011</a> | ✔ CONSIDER making enums with has flags be a power of two.                                       | Warning  |
| <a id="M0012">M0012</a> | ✖ DO NOT have entity types override their base properties.                                      | Error    |
| <a id="M0013">M0013</a> | ✖ DO NOT have complex types override their base properties.                                     | Error    |
| <a id="M0014">M0014</a> | ✔ DO have valid navigation properties for entity sets.                                          | Error    |
| <a id="M0015">M0015</a> | ✔ DO have valid navigation properties for singletons.                                           | Error    |

## Request patterns

| Id                      | Name                                                                                           | Severity |
| ----------------------- | ---------------------------------------------------------------------------------------------- | -------- |
| <a id="H0001">H0001</a> | ✔ DO use `GET …/{collection}` and `GET …/{collection}/{id}` for listing and reading resources. | Error    |
| <a id="H0002">H0002</a> | ✔ DO use `POST …/{collection}` for creating resources.                                         | Error    |
| <a id="H0003">H0003</a> | ✔ DO use `PATCH …/{collection}/{id}` for updating resources.                                   | Error    |
| <a id="H0004">H0004</a> | ✖ AVOID using `PUT …/{collection}/{id}` for updating resources.                                | Warning  |
| <a id="H0005">H0005</a> | ✖ DO NOT use `PATCH` to replaces resources or `PUT` to partially update resources.             | Error    |
| <a id="H0006">H0006</a> | ✖ AVOID patterns that require multiple round trips to complete a single logical action.        | Warning  |
| <a id="H0007">H0007</a> | ✔ CONSIDER supporting `return`, `omit-nulls`, and `include-evolvable-enums` preferences.       | Warning  |
| <a id="H0008">H0008</a> | ✔ DO make requests and responses symmetrical.                                                  | Error    |

## Serialization

| Id                      | Name                                                                   | Severity |
| ----------------------- | ---------------------------------------------------------------------- | -------- |
| <a id="S0001">S0001</a> | ✔ DO use an object as the root of all JSON payloads.                   | Error    |
| <a id="S0002">S0002</a> | ✔ DO use a `value` property in the root object to return a collection. | Error    |
| <a id="S0003">S0003</a> | ✔ DO return a `@odata.context` URL on all responses.                   | Error    |
| <a id="S0004">S0004</a> | ✔ DO include `@odata.type` annotations when the type is ambiguous.     | Warning  |
| <a id="S0005">S0005</a> | ✔ DO return JSON by default.                                           | Error    |
| <a id="S0006">S0006</a> | ✔ DO minify responses.                                                 | Info     |

## Authorization

| Id                      | Name                                                                       | Severity |
| ----------------------- | -------------------------------------------------------------------------- | -------- |
| <a id="A0001">A0001</a> | ✖ DO NOT use a scope ending with `.Read` to authorize a data modification. | Error    |
| <a id="A0002">A0002</a> | ✔ DO use `POST …/{collection}` for creating resources.                     | Error    |
| <a id="A0003">A0003</a> | ✔ DO use `PATCH …/{collection}/{id}` for updating resources.               | Error    |
| <a id="A0004">A0004</a> | ✖ AVOID using `PUT …/{collection}/{id}` for updating resources.            | Warning  |

## Errors

| Id                      | Name                                                                                            | Severity |
| ----------------------- | ----------------------------------------------------------------------------------------------- | -------- |
| <a id="E0001">E0001</a> | ✔ DO return an `error` property with a child `code` property in all error responses.            | Error    |
| <a id="E0002">E0002</a> | ✔ DO return a `403 Forbidden` error when insufficient scopes are present on the auth token.     | Error    |
| <a id="E0003">E0003</a> | ✔ CONSIDER returning a `404 Not found` error if a `403` would result in information disclosure. | Error    |
| <a id="E0004">E0004</a> | ✔ DO return a `429 Too many requests` error when the caller has exceeded throttling limits.     | Error    |
