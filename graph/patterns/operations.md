# Operations

Microsoft Graph API Design Pattern

*The operations pattern provides the ability to model a change that might impact multiple resources and can't be effectively modeled by using HTTP methods.*

## Problem

Sometimes when modeling a complex business domain, API designers need to model a business operation that effects one or multiple resources and has additional semantic meaning that cannot be expressed by HTTP methods. Modeling the operation via HTTP methods on each individual resource might be either inefficient or expose internal implementation details.

## Solution

To address these use cases, API designers can use operational resources such as functions or actions. If the operation doesn't have any side effects and MUST return a single instance of a type or a collection of instances, then the designer SHOULD use OData functions; otherwise, the designer can model the operation as an action.

## When to use this pattern

The operation pattern might be justified when a modeling operation represents one or combination of the following:

- a change of a resource (i.e., increment the value of a property) rather than a state (i.e., the final value of the property)
- complex processing logic that shouldn't be exposed to the client
- operation parameters might convey a restricted set of option (i.e., a report that has to specify a date range)
- the operation leverage some service-side data not exposed to (or easily retrieved in context by) the user.

You can consider related patterns such as [long running operations](./long-running-operations.md) and [change tracking](./change-tracking.md).

## Issues and considerations

- Microsoft Graph does NOT support unbound actions or functions. Bound actions and functions MUST must have the `isBound="true"` attribute and a binding parameter. Bound operations are invoked on resources matching the type of the binding parameter.The first parameter of a bound operation is always the binding parameter.The binding parameter can be of any type, and parameter value MAY be Nullable.

- Both actions and functions support overloading, meaning a schema might contain multiple actions or functions with the same name. The overload rules as per the OData [standard](http://docs.oasis-open.org/odata/odata-csdl-xml/v4.01/odata-csdl-xml-v4.01.html#sec_FunctionOverloads) apply when adding parameters to actions and functions.
  
- Because Microsoft Graph only supports bound actions and functions, all must have at least one parameter where the first is the binding parameter. The MUSTS of parameters are as follows:

  - Each parameter must have a simple identifier name.
  - The parameter name must be unique within the overload.
  - The parameter must specify a type.

- Microsoft Graph supports the use of optional parameters. The optional parameter annotation can be used instead of creating function or action overloads when unnecessary.

- API designer **MUST** use POST to call actions on resources.
- API designer **MUST** use GET to call functions on resources.

- The addition of a new mandatory not-nullable parameter to an existing action or function is a breaking change and is not allowed without proper versioning that is in accordance with our [deprecation guidelines](https://github.com/microsoft/api-guidelines/blob/vNext/graph/deprecation.md).

## Examples

### A user wants to forward email

```
POST https://graph.microsoft.com/v1.0/me/messages/AQMkADNkMmMxYzIwLWJkOTItNDczZC1hNmYyLWUwZjk2ZTljMDQyNQBGAAAD1dY5iRo4x0_pEqop6hOrQAcAeGCrbYV1-kiG-z9Rv6yHMgAAAgEJAAAAeGCrbYV1-kiG-z9Rv6yHMgABRxeUKgAAAA==/forward

{
    "comment": "FYI",
    "toRecipients": [
        {
            "emailAddress": {
                "address": "alex.darrow@microsoft.com",
                "name": "Alex Darrow"
            }
        }
    ]
}
```
Response:
```
HTTP/1.1 202 Accepted

    "cache-control": "private",
    "client-request-id": "ca2d0416-a2c1-05af-df60-0921547a86e9",
    "content-length": "0",
    "request-id": "8b53016f-cc2b-4d9f-9818-bd6f0a5e3cd0"
```

`forward` operation is modeled as an asynchronous action bound to the Graph `message` entity type because the operation represents a complex business logic processed on the server side. 
```
<Action Name="forward" IsBound="true">
        <Parameter Name="bindingParameter" Type="graph.message" />
        <Parameter Name="ToRecipients" Type="Collection(graph.recipient)" />
        <Parameter Name="Message" Type="graph.message" />
        <Parameter Name="Comment" Type="Edm.String" Unicode="false" />
</Action>
```

### A user wants to see recent application activities

```
GET https://graph.microsoft.com/v1.0/me/activities/recent
```

Response:

```
HTTP/1.1 200 OK

{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#Collection(userActivity)",
    "value": []
}
```
`recent` function will query the most recent historyItems and then pull related activities therefore the operation represents a complex business logic processed on the server side. This operation  doesn't change any server data and is a good fit for a function. The function is bound to the collection of `userActivity` entity type.

```
<Function Name="recent" EntitySetPath="activities" IsBound="true">
        <Parameter Name="bindingParameter" Type="Collection(graph.userActivity)" />
        <ReturnType Type="Collection(graph.userActivity)" />
</Function>
```
### Get a report that provides the number of active users using Microsoft Edge

```
https://graph.microsoft.com/beta/reports/getBrowserUserCounts(period='D7')
```

Response:

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 205

{
   "value":[
      {
         "reportRefreshDate":"2021-04-17",
         "reportPeriod":7,
         "userCounts":[
            {
               "reportDate":"2021-04-17",
               "edge":413
            },
            {
               "reportDate":"2021-04-16",
               "edge":883
            }
         ]
      }
   ]
}
```

`getBrowserUserCounts` operation  doesn't change any server data and is a good fit for a function.`period` operation parameter convey a restricted set of options representing the number of days over which the report is aggregated. The report supports only 7,30,90, or 180 days. In addition the function doesn't return a Graph resource but streams response data in JSON or CSV formats.

```
<Function Name="getBrowserUserCounts" IsBound="true">
        <Parameter Name="reportRoot" Type="graph.reportRoot" />
        <Parameter Name="period" Type="Edm.String" Nullable="false" Unicode="false" />
        <ReturnType Type="Edm.Stream" Nullable="false" />
</Function>
```
