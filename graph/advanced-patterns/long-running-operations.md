---
title: Long-running operations
owner: mastaffo
---

# Long-running operations

Long running operations, sometimes called async operations, tend to mean different things to different people. This section sets forth guidance around different types of long running operations, and describes the wire protocols and best practices for these types of operations.

1. One or more clients MUST be able to monitor and operate on the same resource at the same time.
2. The state of the system SHOULD be discoverable and testable at all times. Clients SHOULD be able to determine the system state even if the operation tracking resource is no longer active. The act of querying the state of a long running operation should itself leverage principles of the web. i.e. well defined resources with uniform interface semantics. Clients MAY issue a GET on some resource to determine the state of a long running operation
3. Long running operations SHOULD work for clients looking to "Fire and Forget" and for clients looking to actively monitor and act upon results.
4. Cancellation does not explicitly mean rollback. On a per-API defined case it may mean rollback, or compensation, or completion, or partial completion, etc. Following a cancelled operation, It SHOULD NOT be a client's responsibility to return the service to a consistent state which allows continued service.

## Resource based long running operations (RELO)

Resource based modeling is where the status of an operation is encoded in the resource and the wire protocol used is the standard synchronous protocol. In this model state transitions are well defined and goal states are similarly defined.

_This is the preferred model for long running operations and should be used wherever possible_. Avoiding the complexity and mechanics of the LRO Wire Protocol makes things simpler for our users and tooling chain.

An example may be a machine reboot, where the operation itself completes synchronously but the GET operation on the virtual machine resource would have a "state: Rebooting", "state: Running" that could be queried at any time.

This model MAY integrate Push Notifications.

While most operations are likely to be POST semantics, In addition to POST semantics services MAY support PUT semantics via routing to simplify their APIs. For example, a user that wants to create a database named "db1" could call:

```http
PUT https://api.contoso.com/v1.0/databases/db1
```

In this scenario the databases segment is processing the PUT operation.

Services MAY also use the hybrid defined below.

## Stepwise long running operations

A stepwise operation is one that takes a long, and often unpredictable, length of time to complete, and doesn't offer state transition modeled in the resource. This section outlines the approach that services should use to expose such long running operations.

Service MAY expose stepwise operations.

> Stepwise Long Running Operations are sometimes called "Async" operations. This causes confusion, as it mixes elements of platforms ("Async / await", "promises", "futures") with elements of API operation. This document uses the term "Stepwise Long Running Operation" or often just "Stepwise Operation" to avoid confusion over the word "Async".

Services MUST perform as much synchronous validation as practical on stepwise requests. Services MUST prioritize returning errors in a synchronous way, with the goal of having only "Valid" operations processed using the long running operation wire protocol.

For an API that's defined as a Stepwise Long Running Operation the service MUST go through the Stepwise Long Running Operation flow even if the operation can be completed immediately. In other words, APIs must adopt and stick with a LRO pattern and not change patterns based on circumstance.

### `PUT`

Services MAY enable PUT requests for entity creation.

```http
PUT https://api.contoso.com/v1.0/databases/db1
```

In this scenario the _databases_ segment is processing the PUT operation.

```http
HTTP/1.1 202 Accepted
Location: https://api.contoso.com/v1.0/operations/123
```

For services that need to return a partially created response here, use the hybrid flow described below.

### `POST`

Services MAY enable POST requests for entity creation.

```http
POST https://api.contoso.com/v1.0/databases/

{
  "fileName": "someFile.db",
  "color": "red"
}

HTTP/1.1 202 Accepted
Location: https://api.contoso.com/v1.0/operations/123
```

### `POST` hybrid model

Services MAY respond synchronously to POST requests to collections that create a resource even if the resources aren't fully created when the response is generated. In order to use this pattern, the response MUST include a representation of the incomplete resource and an indication that it is incomplete.

For example:

```http
POST https://api.contoso.com/v1.0/databases/ HTTP/1.1
Host: api.contoso.com
Content-Type: application/json
Accept: application/json

{
  "fileName": "someFile.db",
  "color": "red"
}
```

Service response says the database is being created at the URL provided in the Content-Location header, but indicates the request is not completed by including a 202 status code and Location header pointing to an operation resource. The response body includes the incomplete representation of the resource that will eventually exist at the URL in the Content-Location header.

```http
HTTP/1.1 202 Accepted
Content-Location: https://api.contoso.com/v1.0/databases/db1
Location: https://api.contoso.com/v1.0/operations/123

{
  "databaseName": "db1",
  "color": "red",
  "Status": "Provisioning",
  [ … other fields for "database" …]
}
```

### Operations resource

Services MAY provide an `/operations` resource at the tenant level.

Services that provide the `/operations` resource MUST provide GET semantics. GET MUST enumerate the set of operations, following standard pagination, sorting, and filtering semantics. The default sort order for this operation MUST be:

| Primary Sort           | Secondary Sort          |
| ---------------------- | ----------------------- |
| Not Started Operations | Operation Creation Time |
| Running Operations     | Operation Creation Time |
| Completed Operations   | Operation Creation Time |

::: tip
**Note:** that "Completed Operations" is a goal state (see below), and may actually be any of several different states such as "successful", "cancelled", "failed" and so forth.
:::

### Operation resource

An operation is a user addressable resource that tracks a stepwise long running operation. Operations MUST support GET semantics. The GET operation against an operation MUST return:
1.The operation resource, it's state, and any extended state relevant to the particular API.
2.200 OK as the response code.

Services MAY support operation cancellation by exposing DELETE on the operation. If supported DELETE operations MUST be idempotent.

::: tip
**Note:** From an API design perspective, cancellation does not explicitly mean rollback. On a per-API defined case it may mean rollback, or compensation, or completion, or partial completion, etc. Following a cancelled operation It SHOULD NOT be a client's responsibility to return the service to a consistent state which allows continued service.
:::

Services that do not support operation cancellation MUST return a 405 Method Not Allowed in the event of a DELETE.

Operations MUST support the following states:

1. NotStarted
2. Running
3. Succeeded. Terminal State.
4. Failed. Terminal State.

Services MAY add additional states, such as "Cancelled" or "Partially Completed". Services that support cancellation MUST sufficiently describe their cancellation such that the state of the system can be accurately determined and any compensating actions may be run.

Services that support additional states should consider this list of canonical names and avoid creating new names if possible: Cancelling, Cancelled, Aborting, Aborted, Tombstone, Deleting, Deleted.

An operation MUST contain, and provide in the GET response, the following information:

1. The timestamp when the operation was created.
2. A timestamp for when the current state was entered.
3. The operation state (notstarted / running / completed).

Services MAY add additional, API specific, fields into the operation. The operation status JSON returned looks like:

```json
{
  "createdDateTime": "2015-06-19T12-01-03.45Z",
  "lastActionDateTime": "2015-06-19T12-01-03.45Z",
  "status": "notstarted | running | succeeded | failed"
}
```

### Percent complete

Sometimes it is impossible for services to know with any accuracy when an operation will complete. Which makes using the Retry-After header problematic. In that case, services MAY include, in the operationStatus JSON, a percent complete field.

```json
{
  "createdDateTime": "2015-06-19T12-01-03.45Z",
  "percentComplete": "50",
  "status": "running"
}
```

In this example the server has indicated to the client that the long running operation is 50% complete.

### Target resource location

For operations that result in, or manipulate, a resource the service MUST include the target resource location in the status upon operation completion.

```json
{
  "createdDateTime": "2015-06-19T12-01-03.45Z",
  "lastActionDateTime": "2015-06-19T12-06-03.0024Z",
  "status": "succeeded",
  "resourceLocation": "https://api.contoso.com/v1.0/databases/db1"
}
```

### Operation tombstones

Services MAY choose to support tombstoned operations. Services MAY choose to delete tombstones after a service defined period of time.

### The typical flow, polling

1. Client invokes a stepwise operation by invoking an action using POST
2. The server MUST indicate the request has been started by responding with a 202 Accepted status code. The response SHOULD include the location header containing a URL that the client should poll for the results after waiting the number of seconds specified in the Retry-After header.
3. Client polls the location until receiving a response that indicates the stepwise long running operation is complete.

### Example of the typical flow, polling

Client invokes the restart action:

```http
POST https://api.contoso.com/v1.0/databases HTTP/1.1
Accept: application/json

{
  "fromFile": "myFile.db",
  "color": "red"
}
```

The server response indicates the request has been created.

```http
HTTP/1.1 202 Accepted
Location: https://api.contoso.com/v1.0/operations/123
```

Client waits for a period of time then invokes another request to try to get the operation status.

```http
GET https://api.contoso.com/v1.0/operations/123
Accept: application/json
```

Server responds that results are still not ready and optionally provides a recommendation to wait 30 seconds.

```http
HTTP/1.1 200 OK
Retry-After: 30

{
  "createdDateTime": "2015-06-19T12-01-03.4Z",
  "status": "running"
}
```

Client waits the recommended 30 seconds and then invokes another request to get the results of the operation.

```http
GET https://api.contoso.com/v1.0/operations/123
Accept: application/json
```

Server responds with a "status:succeeded" operation that includes the resource location.

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "createdDateTime": "2015-06-19T12-01-03.45Z",
  "lastActionDateTime": "2015-06-19T12-06-03.0024Z",
  "status": "succeeded",
  "resourceLocation": "https://api.contoso.com/v1.0/databases/db1"
}
```

### The typical flow, push notifications

1. Client invokes a long running operation by invoking an action using POST. The client has a push notification already setup on the parent resource.
2. The service indicates the request has been started by responding with a 202 Accepted status code. The client ignores everything else.
3. Upon completion of the overall operation the service pushes a notification via the subscription on the parent resource.
4. The client retrieves the operation result via the resource URL.

### Example of the typical flow, push notifications existing subscription

Client invokes the backup action. The client already has a push notification subscription setup for db1.

```http
POST https://api.contoso.com/v1.0/databases/db1?backup HTTP/1.1
Accept: application/json
```

The server response indicates the request has been accepted.

```http
HTTP/1.1 202 Accepted
Location: https://api.contoso.com/v1.0/operations/123
```

The caller ignores all the headers in the return.

The target URL receives a push notification when the operation is complete.

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "value": [
    {
      "subscriptionId": "1234-5678-1111-2222",
      "context": "subscription context that was specified at setup",
      "resourceUrl": "https://api.contoso.com/v1.0/databases/db1",
      "userId" : "contoso.com/user@contoso.com",
      "tenantId" : "contoso.com"
    }
  ]
}
```

### `Retry-After`

In the examples above the Retry-After header indicates the number of seconds that the client should wait before trying to get the result from the URL identified by the location header.

The HTTP specification allows the Retry-After header to alternatively specify a HTTP date, so clients should be prepared to handle this as well.

```http
HTTP/1.1 202 Accepted
Location: http://api.contoso.com/v1.0/operations/123
Retry-After: 60
```

::: tip
**Note:** The use of the HTTP Date is inconsistent with the use of ISO 8601 Date Format used throughout this document, but is explicitly defined by the HTTP standard in [RFC 7231][rfc-7231-7-1-1-1]. Services SHOULD prefer the integer number of seconds (in decimal) format over the HTTP date format.
:::

## Retention policy for operation results

In some situations, the result of a long running operation is not a resource that can be addressed. For example, if you invoke a long running Action that returns a Boolean (rather than a resource). In these situations, the Location header points to a place where the Boolean result can be retrieved.

Which begs the question: "How long should operation results be retained?"

A recommended minimum retention time is 24 hours.

Operations SHOULD transition to "tombstone" for an additional period of time prior to being purged from the system
