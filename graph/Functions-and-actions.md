[[_TOC_]]

# Functions and Actions

The [OData standard](http://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html), beside many other things, provides a way to describe the structure of the requests and responses of an OData service via the [Common Schema Definition Language](http://docs.oasis-open.org/odata/odata-csdl-xml/v4.01/odata-csdl-xml-v4.01.html) (CSDL).

## Functions

The standard [defines](http://docs.oasis-open.org/odata/odata-csdl-xml/v4.01/odata-csdl-xml-v4.01.html#sec_Function) functions as:
>service-defined operations that MUST NOT have observable side effects and MUST return a single instance or collection of instances of any type.

Please see the OData standard page to get more details on functions.

### Example

```XML
<Function Name="getNumber" IsBound="true">
  <Parameter Name="name" Type="Microsoft.graph.name"/>
  <ReturnType Type="Edm.Int32"/>
</Function>
```

## Actions

The standard [defines](http://docs.oasis-open.org/odata/odata-csdl-xml/v4.01/odata-csdl-xml-v4.01.html#sec_Action) actions as:
>service-defined operations that MAY have observable side effects and MAY return a single instance or a collection of instances of any type.

Please see the OData standard for more details on actions.

### Example

```XML
  <Action Name="createNumber" IsBound="true">
    <Parameter Name="name" Type="Microsoft.graph.name" />
    <Parameter Name="number" Type="Edm.Int32" />
    <ReturnType Type="Microsoft.graph.personNameCombo" />
  </Action>
```

## Bound vs Unbound

MS Graph does NOT support unbound actions or functions.

Bound actions and functions are invoked on resources matching the type of the binding parameter. The binding parameter can be of any type, and it MAY be Nullable. For MS Graph, actions and functions must have the `isBound="true"` attribute. The first parameter is the binding parameter.

## Overloads

Both actions and functions support overloading, meaning a schema may contain multiple actions or functions with the same name. See OData [standard](http://docs.oasis-open.org/odata/odata-csdl-xml/v4.01/odata-csdl-xml-v4.01.html#sec_FunctionOverloads) for more information and rules.

## Parameters

As Graph only supports bound actions and functions, all must have at least one parameter where the first is the binding parameter. The MUSTS of parameters:

- Each parameter must have a simple identifier name.
- The parameter name must be unique within the overload.
- The parameter must specify a type.

The overload rules as per the OData [standard](http://docs.oasis-open.org/odata/odata-csdl-xml/v4.01/odata-csdl-xml-v4.01.html#sec_FunctionOverloads) apply when adding parameters to actions and functions.

### Optional Parameters

Graph supports the use of optional parameters. The optional parameter annotation can be used instead of creating function or action overloads when unnecessary. 

Example:
The `getNumber` function has an optional parameter `date` . You can use an optional parameter with this annotation:

```XML
<Function Name="getNumber" IsBound="true">
  <Parameter Name="name" Type="Microsoft.graph.name"/>
  <Parameter Name="date" Type="Edm.DateTimeOffset">
    <Annotation Term="Org.OData.Core.V1.OptionalParameter"/>
  </Parameter>
  <ReturnType Type="Edm.Int32"/>
</Function>
```

instead of using an overload like this:

```XML
<Function Name="getNumber" IsBound="true">
  <Parameter Name="name" Type="Microsoft.graph.name"/>
  <ReturnType Type="Edm.Int32"/>
</Function>

<Function Name="getNumber" IsBound="true">
  <Parameter Name="name" Type="Microsoft.graph.name"/>
  <Parameter Name="date" Type="Edm.DateTimeOffset"/>
  <ReturnType Type="Edm.Int32"/>
</Function>
```