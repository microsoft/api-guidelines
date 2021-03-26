[[_TOC_]]

# RFC: Omitting properties

> Warning<br/>Review period ends 10/10/2018<br/>
> <https://github.com/Microsoft/graph-onboarding/pull/2>

There are scenarios where the server contains business logic that determines if a property value should be returned, or not, to the client. Even when the client explicitly requests the property, it may be purposefully omitted from the response by the server. This section provides guidance on how the server should explicitly represent omitted properties in the response.

> TIP<br/>
> In such scenarios, the property is publicly known and published through the schema. Clients are aware of the property and they can request it, but the server decides to not return it.

## Scenarios

These are scenarios existing today where omitting properties is desirable:

- The property value is only available in tenants that subscribe to a specific level of paid product, such as Azure AD Premium P2 or Microsoft 365 E5.
- The client is calling with permissions that are not sufficient to view the property. For example, `User.ReadBasic.All` allows access to only the basic properties of the `User` entity.
- The client is calling on behalf of a user whose relationship to the target entity allows limited access. For example, a teacher is accessing student data, but the student is not in the teacher's classroom, so limited properties of the student entity should be returned.

## `omitted` annotation

Returning `null` values or implicitly hiding properties creates ambiguity and does not promote correct app logic. Instead, when the server decides to omit property values, it should explicitly state that properties are omitted in the response.

When a property is omitted, it should still be returned with a `null` value; this allows client code that expects the property to work seamlessly and makes the handling of the annotations optional.

In addition to the null property, a corresponding annotation should be included using the following format:

```json
"{propertyName}@omitted":
{
  "code": "{code}"
}
```

- `propertyName` matches the name of the original property. It is followed by the annotation `@omitted`.
- The value is a JSON payload with one property - `code` - whose value is one of the predefined "reason codes". This value can be interpreted programatically, and it is also human-readable.

App code can interpret this portion of the response to pivot its business logic. For example, a developer can create a multi-tenant application that works in all tenants, even ones that never have access to certain property values; the app can detect when values are omitted and react as appropriate.

### Response example

An app is getting a resource that is a collection of `sampleEntity`. It selects properties 1 to 3.

```http
GET
https://graph.microsoft.com/beta/sampleEntities?$select=id,property1,property2,property3
```

Let's assume that in the tenant in which this request is made, `property3` is not available because the tenant does not have the required product license. The response would look as follows:

```http
{
  "@odata.context": "https://graph.microsoft-ppe.com/v1.0/$metadata#sampleEntities",
  "value": [
    {
      "id": "guidA",
      "property1": "valueA-1",
      "property2": "valueA-2",
      "property3": null,
      "property3@omitted": {
        "code": "licensedProductRequired"
      }
    },
    {
      "id": "guidB",
      "property1": "valueB-1",
      "property2": "valueB-2",
      "property3": null,
      "property3@omitted": {
        "code": "licensedProductRequired"
      }
    },
  ]
}
```

### Code values

Code values should be well defined and documented. There should be a unique code for each major scenario that is common across the entire Graph API surface. Teams should strive to align their case with more generic global scenarios.

These are the code values based on existing scenarios exposed through Graph today:

| Code                    | Scenario                                                                                                              |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------- |
| licensedProductRequired | A licensed product is required in the tenant, or a license must be assigned to the user whose data is being accessed. |
| limitedPermissions      | The app or user permissions used to make the call are insufficient to access the specific property.                   |
| limitedRole             | The role of the app or user in relation to the target entity is insufficient to access the specific property.         |

> TIP<br/>
> The difference between `limitedPermissions` and `limitedRole` is subtle, but important. The former will result in an omitted property for all entities in a collection. For example, using the `User.ReadBasic.All` permission will omit properties from **all** users returned. The latter may result in properties omitted for only some entities. For example, a teacher reading users in a school may see full properties for students in their classroom, while seeing limited properties for the rest of the students.

### When to return annotations

`@omitted` annotations should be returned when a property was in scope for the requests, but the server decided to omit it. For a property to be in scope means either of these two cases:

- the request included the `$select` parameter that referenced the property
- or, the request did not include `$select` but the property would normally be included as part of the default property set for the entity (note: some workloads include a subset of properties by default, while others include all properties).

## Callers opt-in to this behavior

Unless the caller explicitly opts-in into omit annotations, the response should simply ignore the property and not include it in the response. Only when the specific header value is included in the request, should the behavior described above kick in.

The header used to opt-in is as follows:

```http
TBD - we have not finalized the header
```
