Change Tracking {align="center" style="text-align:center"}
============

Microsoft Graph API Design Pattern

Change tracking enables users to keep a third party system in sync with changes in Microsoft Graph without having to continuously poll the API.

Context
-------

In the world of distributed systems and richness of devices, keeping data in sync across systems is paramount to delivering a great user experience.

* *

Problem:

Allowing customers to keep data in sync on a third party system can be complex and lead to exposing internal service architecture/implementation details. Examples of such details that can be challenging to both sides implementers include:

- Partitions management and requests affinity -> two external requests can fall on different replicas that don't have the same sync state.
- Entities updates tracking -> the way entities are sync internally and tracking mechanisms should not "leak" externally. (update id, update date, conflicts resolution...)
- Multiple sync destinations -> having multiple systems syncing to the same dataset should not impact behavior/performance of the system.

* *

Solution
--------

Change Tracking (aka delta) provides developers with a new endpoint they can use to sync their third party system, get a delta link with a watermark, go offline, and comeback with that delta link to get new changes since their last request. Delta guarantees integrity of data through the watermark, regardless of service partitions and other obscure aspects for clients.

* *

Issues and Considerations
-------------------------

Implementer MUST implement a watermark storage system in case of active watermarks.

When to Use this Pattern
------------------------

Use this pattern when you want to provide customers with the ability to sync the data to a third party system.
Avoid this pattern when customers want to get notified of changes, or for backup/export scenarios.

Example
-------

A good example of such scenario could be the typical contacts list stored in Microsoft 365 services and available in multiple client applications. Obviously having all these client applications continuously polling the service would be suboptimal and deliver a poor end user experience (sync across service partitions without sticky sessions etc...). Using change tracking clients are able to sync contacts, go offline, and get new changes when they come back online.