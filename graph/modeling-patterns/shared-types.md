---
title: Shared types
owner: mastaffo
---

# Working with shared types aka referencing existing models from your schema

You will frequently need to connect your models to other entities on the Graph (indeed, this is a key value of the Microsoft Graph).
To do this, you will have to provide reference versions of the entities you need in your own schema.

For each entity you need to refer to:

- Add a declaration to your own schema. This will live under your own namespace, so refer to it as though it was in your own namespace
- Omit the `ags:IsMaster` annotation to indicate that your service does not master this entity
- Do not express inheritance relationships between external entities
- Do not include a copy of the 'virtual' entity named `entity` - this is synthesized by Microsoft Graph itself

Duplicate enumerations and complex types - these must be expressed (and identical) in every workload that shares them.
