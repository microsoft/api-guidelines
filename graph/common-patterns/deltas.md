---
title: Adding support for delta queries
owner: vibiret
---

# Adding support for delta queries

Delta query enables application to discover newly created, updated, or deleted entities without performing a full read of the target resource with every request. Microsoft Graph applications can use delta query to efficiently synchronize changes with a local data store. For an overview of the general concept, please [refer to the public documentation](https://docs.microsoft.com/en-us/graph/delta-query-overview).

## Why should you add delta query support for your entities?

There are different scenarios where customers are looking at syncing data to a separate system and/or tracking changes in a non-lossy way (making sure they are not missing any changes). These scenarios include compliance solutions, DLP solutions, apps that need to support offline usage and many more.
Today, if your API surface does not support delta queries, the only avenue for customers to implement such scenarios is by **continuously query your API surface**. This increases the complexity and cost of such solutions for customers or makes implementing certain scenarios impossible at scale. More importantly, it greatly increases COGS for the Microsoft Graph as well as your API.

The Microsoft Identity Platform (AAD) has implemented delta query support for a majority of it's entities and will continue to deliver more delta query support in an effort to provide a better experience for customers but also to **decrease COGS**.

## How to add support for delta queries in your API

### Create a new API onboarding review item

Because you'll edit the API schema, you need to go through API review. Go ahead and create [an API review work item](https://microsoftgraph.visualstudio.com/onboarding/_workitems/create/API%20Review).

> Note: if you are adding a net new API with Delta query support on day one, you can reuse the existing API onboarding review item so long as the delta query support was included in the initial API review

### Update the public documentation

Both the API review process and the API schema modification process will require you to provide a link for documentation update before allowing your changes to be added. By creating the public documentation ahead of time, you're making sure you have the required elements ahead of time and won't be blocked.
There are a few places where the reference of a new delta query support must be inserted.

1. You need to add your resource to the [table of supported resources](https://docs.microsoft.com/en-us/graph/delta-query-overview#supported-resources)
1. You need to state that your resource supports delta query in the abstract, eg [orgContact](https://docs.microsoft.com/en-us/graph/api/resources/orgcontact?view=graph-rest-1.0). (\*)
1. You need to add a delta query support page for the resource in the api reference eg [orgContact delta](https://docs.microsoft.com/en-us/graph/api/orgcontact-delta?view=graph-rest-1.0&tabs=http). (\*)
1. You need to add reference to any page you added in the coresponding Table Of Content.
1. You need to add an entry for each version/entity that supports Delta queries to the [changelog](https://docs.microsoft.com/en-us/graph/changelog), [guidance](../../document/guidelines/changelog.html)

> \*: These pages are available for beta and v1.0, make sure you update the beta pages during the public preview of change notifications support for your API. Make sure you update v1.0 pages when support ships for general availability. Updates for different versions can be done in different pull request.

You can see an example of adding delta query support for both v1.0 and beta to the docs [here](https://github.com/microsoftgraph/microsoft-graph-docs/pull/7451).

Once your documentation pull request is submitted, the PR must be labeled with "Do not merge" until the changes are in place on the service, and then change the label to "Ready to merge". You should keep the link to the pull request at hand, you'll need it for the next steps.

Should you require assistance with the documentation process, you can contact the [docs V-Team](mailto:MSGraphDocsVteam@microsoft.com).

### Submit your API for review

Now that you have pre-requisite items, you are ready to submit an [API review](../../review/final-prep.html). Describe that you are adding support for delta query (function) for your API.

> Note: if you are adding a net new API with Delta query support on day one, you can reuse the existing API onboarding review so long as the delta query support was included in the initial API review

### Update the schema metadata

Once your API review has been completed, you are ready to publish the schema changes to indicate your Entity Type support delta queries.

For this step you'll need:

- The link to the API review work item previously created
- The link to the API review pull request previously completed
- The link to the documentation pull request (you can use the same link for docs and changelog changes)

In your schema file you need to add the new delta function. Here is an example for the orgContact entity. (this needs to be added as a child of the Schema tag)

```xml
<Function Name="delta" IsBound="true">
    <Parameter Name="bindingParameter" Type="Collection(Microsoft.DirectoryServices.orgContact)" />
    <ReturnType Type="Collection(Microsoft.DirectoryServices.orgContact)" />
</Function>
```

In your entity declaration, you need to add an annotation stating that the entity supports delta query. (this needs to be added as a child of the EntityType tag)

```xml
<Annotation Term="Org.OData.Capabilities.V1.ChangeTracking">
  <Record>
    <PropertyValue Property="Supported" Bool="true" />
  </Record>
</Annotation>
```

You can now submit the schema changes following the [guidance](https://msazure.visualstudio.com/One/_wiki/wikis/Microsoft%20Graph%20Partners/55110/Test-Using-VSTS-Repo).

> Note: if you want the delta capability to be hidden from publicly available APIs for testing reasons, you can leverage [Privileged identities](https://msazure.visualstudio.com/One/_wiki/wikis/Microsoft%20Graph%20Partners/55117/Privileged-Api) by setting the `ags:IsHidden="true"` attribute on both the annotation and the function.

### Provide required information to support

In order to provide proper support for our customers, the support teams need information you need to provide them with.

This information will be provided by starting a separate process owned by support called SPOT. To start the process, [create a new intake](https://microsoftspot.azurewebsites.net/Intake). A release manager will then contact you and guide you through the process of collecting and documenting the required information for support teams.

When creating the SPOT intake, make sure you indicate you are adding delta query support for your workload and set the following fields:

- Disclosure level: No restrictions
- Release Type: Product/Service/Program
- Release Sub Type: Feature
- Responsible Org: Deployment Services
- Responsible Team: DS C+AI Team

In relevant links, add the [delta query support wiki](https://supportability.visualstudio.com/AzureAD/_wiki/wikis/AzureAD/311338/Microsoft-Graph-Notifications-and-Change-Tracking-using-webhooks) `https://supportability.visualstudio.com/AzureAD/_wiki/wikis/AzureAD/311338/Microsoft-Graph-Notifications-and-Change-Tracking-using-webhooks`

Some of the information that support will require to add it to their internal documentation include:

- The ICM service and team for escalation
- The owning team (distribution list)

### Implement query response

You need to implement a query response to the `/microsoft.graph.delta` (also aliased `/delta`) requests that the Aggregator Gateway Service (AGS, the service immediately behind graph.microsoft.com) will forward to your workload's API.

> The delta query endpoint should match the following pattern to avoid requiring code changes for request transformation in the AGS: `/{version}/entity/delta`.

> The delta query endpoint should authorize on the same permissions required to enumerate the entity type.

The [OData ASP.NET](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.OData/) and [OData ASP.NET core](https://www.nuget.org/packages/Microsoft.AspNetCore.OData) provide base controllers, serialization wrappers and more that help you generate the delta query response as shown [in the public documentation](https://docs.microsoft.com/en-us/odata/webapi/deltafeed_support).

### Update routing information

If your delta query implemention does not live on the same service (FQDN) as your entity's API, you need to update the endpoint routing configuration to account for it. For more information, see [Gradual Config Rollout ACIS Operations](https://msazure.visualstudio.com/One/_wiki/wikis/Microsoft%20Graph%20Partners/40069/Gradual-Config-Rollout-ACIS-Operations).

You also need to update your workload configuration to indicate your support the delta sync protocol eg:

```xml
<ServiceProperties>
  <Items>
    <KeyValuePairItem Value="ODataV4" Key="DeltaSyncProtocol">
      <Overrides />
    </KeyValuePairItem>
    <!-- ... -->
  </Items>
</ServiceProperties>
```

### Handle delta and skip tokens

#### Skip tokens

Skip tokens allow you to implement pagination for the response result to avoid returning too many results at once potentially leading to long response times.

You must return a @odata.nextLink property with the response object if the enumeration contains more elements than requested (when \$top is provided by the customer, or default number of changes to return).

The nextLink is a string that can contain any information you choose and it should contain a deltaLink that represents the starting point of the initial request as well as information to skip elements that have already been enumerated.

When a nextLink is returned, you must not return a deltaLink. The nextLink will be encoded and prefixed by the public Microsoft Graph URL automatically by the AGS before being returned to the client. The client will use the public nextLink to:

1. Know more changes must be enumerated.
1. Query the next changes.

When querying the the public nextLink, the AGS will decode the nextLink (value of \$skiptoken query parameter) that was provided by the workload and provide the decoded value as a query parameter named `nextLink` to the workload when forwarding the request.

When all changes are enumerated, a @odata.deltaLink property should be attached to the response object.

#### Delta tokens

Delta tokens allow you to watermark the last change seen by client so you can know exactly where the client left off on the change feed. You can then present to the client on their next request the changes that haven't already been seen. Under no circumstances should a client be missing changes between two delta requests (no gaps).

If a response to a delta query request is returning the last change available at the time, a @odata.deltaLink property must be added to the response object. The deltaLink is a string that can contain any information you choose and it should allow you to precisely identify the last change seen by the client.

When a deltalink is returned, you must not return a nextLink. It signals to the client that all the current changes have been enumerated and that they should query back, with the deltaLink, at a later time. The deltaLink will be encoded and prefixed by the public Microsoft Graph URL automatically by the AGS before being returned to the client.

When querying the the public deltaLink, the AGS will decode the deltaLink (value of \$deltatoken query parameter) that was provided by the workload and provide the decoded value as a query parameter named `deltaLink` to the workload when forwarding the request.

> Delta links and next links should not exceed 10k characters to avoid routing issues at the AGS level.

### Handle filter, top and select query paremeters

You should consider how these Odata query parameters may or may not be supported for optimizing the response:

- **\$select**: you **must** support the select query parameter to client applications to filter which properties they'd like to get in the response.
- **\$filter**: you may support the filter query parameter to allow client applications to filter which objects they'd like to get from the response.
- **\$top**: you may support the top query parameter to allow client applications to customize the number of results they'd like to get per page in the response. (see nextLink)
- **\$expand**: you may support the expand query parameter to allow client applications to get additional linked entities they get in the response.
- **\$oderby**: you may support the orderby query parameter to allow client applications to customize the order of the results they get in the response.
- **\$skip**: you should **not** support the skip query parameter as it's behavior might conflict with with nextLink behavior already madated by delta query.
- **\$count**: you may support the count query parameter to allow clients applications to get the count of items in the response along with the results.
- **\$search**: you may support the search query parameter to allow clients applications to filter which objects they'd like to get from the response.
- **\$format**: you should **not** support the format query parameter as the AGS is doing some data parsing and replacement before returing the response to clients applications and supports limited formats (JSON).

> If you are building your API using ASP.NET MVC or ASP.NET core MVC you can leverage the OData library to parse and apply oData query paremeters to the delta feed. Make sure you [configure your service pipeline and add the enablequery tag](https://docs.microsoft.com/en-us/odata/webapi/first-odata-api). The library also allows you to [advertise non-support of some query parameters on object properties](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnet.odata.query.nonfilterableattribute?view=odata-aspnetcore-7.0) via attributes.

> If you want more control over OData query options, you can additionally leverage the [ODataQueryOptions object](https://docs.microsoft.com/en-us/aspnet/web-api/overview/odata-support-in-aspnet-web-api/supporting-odata-query-options#invoking-query-options-directly).

> Odata query parameters will be added in the encoded delta/skip token provided to the client application so they do not have to add it to each request. They parameters will be decoded and provided to your workload API by the AGS in any subsequent request. Updating the OData query parameters is not supported after an initial delta/skip token has been generated. The parameters stay consistent over time or the client application must restart the synchronization from scratch with the new parameters (querying delta API with no delta/skip token).

## How to get help

Should you need any help during your design and implementation, there are a couple of ways you can reach out:

- [Stackoverflow.com](https://stackoverflow.com): for any question that does not contain confidential, internal or customer related information. Example: questions about ASP.NET core MVC, questions about the OData libraries, etc.
- [Internal Stackoverflow](https://stackoverflow.microsoft.com): for any question that might contain confidential or internal information. Example: how do I configure the routing in AGS to do...
- [Teams: Microsoft Graph > Delta query](https://teams.microsoft.com/l/channel/19%3a32dabdaf736a4c9482fc3d96967b2ea8%40thread.skype/Delta%2520query?groupId=6d279915-6f5d-452c-895b-7f4f82038843&tenantId=72f988bf-86f1-41af-91ab-2d7cd011db47): for any question that contains details really specific to your implementation, escalation of questions left unanswered on one of the stackoverflow platforms, ...

Ask: please refrain from asking questions directly to the engineering team via either emails, Teams chat etc... This does not scale and it doesn't capture the question (and answer) for other people that might have the same question as you. You should always try to ask a question on stack overflow first before reaching out on the Teams channel.

## End to end testing

You can use canary and ppe for end to end testing of the implementation as outlined in [the following documentation](https://msazure.visualstudio.com/One/_wiki/wikis/Microsoft%20Graph%20Partners/55110/Test-Using-VSTS-Repo).
Private previews can be done by creating a _new version_ of Microsoft Graph as outlined in the documentation previously linked.
