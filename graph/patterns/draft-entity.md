# "Draft" Entity

Microsoft Graph API Design Pattern

*A "draft" entity is one which can be used to temporarily store a subset of an entity that it is a draft of, avoiding validations that "real" instances of the entity would normally require.*

## Problem

There are situations when a client only has a portion of the data required to create an entity at a given time. 
In these situations, the client needs to store that partial data somewhere without requiring all of the data, and with allowing potentially invalid data to be present.

## Solution

The solution is to have a "draft" entity for the desired entity. 
This "draft" can then be used to store the partial data until all of the data is available, at which time the "draft" can be used to create a "real" entity instance. 
Doing this allows the storage of the partial data without compromising the integrity of the model for the "real" entity.

```xml

```

## When to use this pattern

The "draft" entity pattern will be useful for workloads that are backing a UI and expose entities which the UI creates through the use of a multi-page process. 

## Issues and considerations

This pattern should be avoided; instead, use some storage external to graph. 
This pattern *may* be used as a stop-gap if a workload does not yet have the infrastructure for external storage; these entities should be deprecated immediately since they are only a stop-gap.

## Example

*Provide a short example from real life.*