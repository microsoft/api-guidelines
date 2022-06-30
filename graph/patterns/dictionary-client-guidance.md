# Dictionary types

> **Note:** This document will be moved into a central client guidance document in the future.

*The client guidance is a collection of additional information provided to SDK implementers and client applications. This information is meant to help understand how various guidelines and concepts translate in their world and clarify a few unknowns. Always read the corresponding guideline first to get a contextual understanding.*

For more information, see the [Dictionary](./dictionary.md) pattern.

## OpenAPI example

The following json-schema/OpenAPI example defines a dictionary of which values are of type **RoleSettings**.

In **components** in **schemas**:

```json
{
  "roleSettings": {
    "type": "object",
      "properties": {
        "domain": {
          "type": "string"
        }
      }
    }
  }
}
```

```json
{
  "type": "object",
  "patternProperties": {
    ".*": {
      "$ref": "#/components/schemas/roleSettings"
    },
    "additionalProperties": false
  }
}
```

## SDK support

SDKs need to provide support for dictionary types so that SDK consumers get a delightful development experience. Examples are provided for different languages. Other aspects need to be taken into consideration:

- Dictionaries support OData annotations (values prefixed with **@OData**); such annotations should not be inserted directly in the dictionary but rather in the additional properties manager.
- Dictionary types can inherit another dictionary type; this inheritance must be respected.
- Dictionary values can be of union types; if the target language doesn't support union types, a wrapper type should be generated as a backward compatible solution with properties for each type of the union.

### Dotnet

```CSharp
Dictionary<string, RoleSettings>
```

### Java

```Java
Map<string, RoleSettings>
```

### JavaScript/TypeScript

```TypeScript
Map<string, RoleSettings>
```

or

```JavaScript
{
  [key: string]: {value: RoleSettings}
}
```

## Request builder generation annotation

By default, SDKs are not required to contain a set of request builders to run CRUD requests on entries in the dictionary. The dictionary is updated as a whole by consumers by sending requests to the parent entity.

If a **SupportedHttpMethod** annotation is specified for the dictionary type, request builders should be generated to allow consumers to automatically update the entries.
