# Error condition responses

For non-success conditions, developers SHOULD be able to write one piece of code that handles errors consistently across different Microsoft REST API services.
This allows building of simple and reliable infrastructure to handle exceptions as a separate flow from successful responses.
The following is based on the OData v4 JSON spec.
However, it is very generic and does not require specific OData constructs.
APIs SHOULD use this format even if they are not using other OData constructs.

The error response MUST be a single JSON object.
This object MUST have a name/value pair named "error". The value MUST be a JSON object.

This object MUST contain name/value pairs with the names "code" and "message", and it MAY contain name/value pairs with the names "target", "details" and "innererror."

The value for the "code" name/value pair is a language-independent string and MUST match the HTTP response status code description, converted to camelCase, as listed in the [Status Code Registry (iana.org)](https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml)
For example, if the HTTP status code is "Not Found", then the "code" value MUST be "notFound".

Most services will require a larger number of more specific error codes, which are not interesting to all clients.
These error codes SHOULD be exposed in the "innererror" name/value pair as described below.
Introducing a new value for "code" that is visible to existing clients is a breaking change and requires a version increase.
Services can avoid breaking changes by adding new error codes to "innererror" instead.

The value for the "message" name/value pair MUST be a human-readable representation of the error.
It is intended as an aid to developers and is not suitable for exposure to end users.
Services wanting to expose a suitable message for end users MUST do so through an [odata-json-annotations](https://docs.oasis-open.org/odata/odata-json-format/v4.01/cs02/odata-json-format-v4.01-cs02.html#sec_AnnotateaJSONObject) or custom property.
Services SHOULD NOT localize "message" for the end user, because doing so might make the value unreadable to the app developer who may be logging the value, as well as make the value less searchable on the Internet.

The value for the "target" name/value pair is the target of the particular error (e.g., the name of the property in error).

The value for the "details" name/value pair MUST be an array of JSON objects that MUST contain name/value pairs for "code" and "message", and MAY contain a name/value pair for "target", as described above.
The objects in the "details" array usually represent distinct, related errors that occurred during the request.
See example below.

The value for the "innererror" name/value pair MUST be an object.
The contents of this object are service-defined.
Services wanting to return more specific errors than the root-level code MUST do so by including a name/value pair for "code" and a nested "innererror". Each nested "innererror" object represents a higher level of detail than its parent.
When evaluating errors, clients MUST traverse through all of the nested "innererrors" and choose the deepest one that they understand.
This scheme allows services to introduce new error codes anywhere in the hierarchy without breaking backwards compatibility, so long as old error codes still appear.
The service MAY return different levels of depth and detail to different callers.
For example, in development environments, the deepest "innererror" MAY contain internal information that can help debug the service.
To guard against potential security concerns around information disclosure, services SHOULD take care not to expose too much detail unintentionally.
Error objects MAY also include custom server-defined name/value pairs that MAY be specific to the code.
Error types with custom server-defined properties SHOULD be declared in the service's metadata document.
See example below.

Error responses MAY contain Odata JSON annotations in any of their JSON objects.

We recommend that for any transient errors that may be retried, services SHOULD include a Retry-After HTTP header indicating the minimum number of seconds that clients SHOULD wait before attempting the operation again.

## ErrorResponse : Object

Property | Type | Required | Description
-------- | ---- | -------- | -----------
`error` | Error | ✔ | The error object.

## Error : Object

Property | Type | Required | Description
-------- | ---- | -------- | -----------
`code` | String | ✔ | One of a server-defined set of error codes.
`message` | String | ✔ | A human-readable representation of the error.
`target` | String |  | The target of the error.
`details` | Error[] |  | An array of details about specific errors that led to this reported error.
`innererror` | InnerError |  | An object containing more specific information than the current object about the error.

## InnerError : Object

Property | Type | Required | Description
-------- | ---- | -------- | -----------
`code` | String |  | A more specific error code than was provided by the containing error.
`innererror` | InnerError |  | An object containing more specific information than the current object about the error.

## Examples

Example of "innererror":

```json
{
  "error": {
    "code": "unauthorized",
    "message": "Previous passwords may not be reused",
    "target": "password",
    "innererror": {
      "code": "passwordError",
      "innererror": {
        "code": "passwordDoesNotMeetPolicy",
        "minLength": "6",
        "maxLength": "64",
        "characterTypes": ["lowerCase","upperCase","number","symbol"],
        "minDistinctCharacterTypes": "2",
        "innererror": {
          "code": "passwordReuseNotAllowed"
        }
      }
    }
  }
}
```

In this example, the most basic error code is "unauthorized", but for clients that are interested, there are more specific error codes in "innererror."
The "passwordReuseNotAllowed" code may have been added by the service at a later date, having previously only returned "passwordDoesNotMeetPolicy."
Existing clients do not break when the new error code is added, but new clients MAY take advantage of it.
The "passwordDoesNotMeetPolicy" error also includes additional name/value pairs that allow the client to determine the server's configuration, validate the user's input programmatically, or present the server's constraints to the user within the client's own localized messaging.

Example of "details":

```json
{
  "error": {
    "code": "badRequest",
    "message": "Multiple errors in ContactInfo data",
    "target": "contactInfo",
    "details": [
      {
        "code": "nullValue",
        "target": "phoneNumber",
        "message": "Phone number must not be null"
      },
      {
        "code": "nullValue",
        "target": "lastName",
        "message": "Last name must not be null"
      },
      {
        "code": "malformedValue",
        "target": "address",
        "message": "Address is not valid"
      }
    ]
  }
}
```

In this example there were multiple problems with the request, with each individual error listed in "details."
