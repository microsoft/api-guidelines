
# Antipattern name

*name with a negative connotation*

*Example: Flatbag of properties* 

## Description

*Example: The flat bag pattern is a known anti-pattern in Microsoft Graph, where multiple variants of a common concept are modeled as a single entity type with all potential properties plus an additional property to distinguish the variants.*  

## Consequences
*Describe the consequences in terms of the developer experience*

*Example: This is the least recommended modeling choice because it is weakly typed, which increases the number of variations and complexity of solutions, making it difficult to verify the semantic correctness of the API for both clients and producers...* 

## Preferable solutions 

*Example: It is preferable to use type hierarchy and facets patterns.
Should provide a link to a valid pattern or patterns.* 

## Example

*Provide an example of better modeling*
