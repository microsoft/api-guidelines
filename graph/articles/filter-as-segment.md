# Filter as segment

There is an [OData feature](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part2-url-conventions.html#sec_AddressingaSubsetofaCollection) which allows having a `$filter` in a URL segment. 
This feature is useful whenever there are operations on a collection and the client wants to perform those operations on a *subset* of the collection. 
For example, the `riskyUsers` API on Microsoft Graph has an action defined to let clients "dismiss" risky users (i.e. consider those users "not risky"):

```xml
<Action Name="dismiss" IsBound="true">
  <Parameter Name="bindingParameter" Type="Collection(microsoft.graph.riskyUser)" />
  <Parameter Name="userIds" Type="Collection(Edm.String)" />
</Action>
```

Using this action, clients can call

```http
POST /identityProtection/riskyUsers/dismiss
{
  "userIds": [
    "{userId1}",
    "{userId2}",
    ...
  ]
}
```

in order to dismiss the risky users with the provided IDs. Using the filter-as-segment OData feature, the action could instead be defined as:

```xml
<Action Name="dismiss" IsBound="true">
  <Parameter Name="bindingParameter" Type="Collection(self.riskyUser)" />
</Action>
```

and clients could call

```http
POST /identityProtection/riskyUsers/$filter=@f/dismiss?@f=id IN ('{userId1}','{userId2}',...)
```

Doing this is beneficial due to the robust nature of OData filter expressions: clients will be able to dismiss risky users based on any supported filter without the service team needing to implement a new `dismiss` overload that filters based on the new criteria.
However, there are some concerns about the discoverability of using the filter-as-segment feature, as well as the support of [parameter aliasing](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part2-url-conventions.html#sec_ParameterAliases) that's required.
As a result, functions should be introduced that act in the same way as the filter-as-segment:

```xml
<Function Name="filter" IsBound="true" IsComposable="true">
  <Parameter Name="bindingParameter" Type="Collection(microsoft.graph.riskyUser)" Nullable="false" />
  <Parameter Name="expression" Type="Edm.String" Nullable="false" />
  <ReturnType Type="Collection(microsoft.graph.riskyUser)" />
</Function>
```

Clients would now be able to call

```http
POST /identityProtection/riskyUsers/filter(expression='id IN (''{userId1}'',''{userId2}'',...)')/dismiss
```

NOTE: the `'` literal in the filter expression must be escaped with `''`

An example implementation of a filter function using OData WebApi can be found [here](https://github.com/OData/AspNetCoreOData/commit/7732f7e6b812d9a79a73529562f2e74b68e2794f).
