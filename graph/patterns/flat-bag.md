# Flat Bag Pattern

Microsoft Graph API Design Pattern

### *A known pattern in Microsoft Graph is to model multiple variants of a common concept as a single entity type with all the potential properties plus an additional property to distinguish the variants.*


## Problem

API designer needs to model a small and limited number of variants of a common concept with a concise list of non-overlapping properties and consistent behavior across variant. The designer also wants to simplify query construction.

## Solution

The API designer creates one entity type with all the potential properties plus an additional property to distinguish the variants, often called `variantType`.

## Issues and Considerations

????


## When to Use this Pattern

The flat-bag pattern is useful when there is a small number of variants 

????

There are related patterns to consider such as
[Type Hierarchy](https://github.com/microsoft/api-guidelines/tree/graph/graph) and [Flat
bag of
properties](https://github.com/microsoft/api-guidelines/tree/graph/graph).

## Example
?????
