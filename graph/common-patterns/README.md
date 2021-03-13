---
title: Common API Patterns
owner: vibiret
---

# Common patterns to implement in your API

Great APIs provide consistent ways to address common problems. The common problems include:

- The ability to get notified (push) when a change occurs in the data exposed by Microsoft Graph. This is addressed implementing [change notifications (aka webhooks)](./webhooks.md).
- The ability to track changes (pull) occuring in the data exposed by Microsoft Graph. This is addressed implementing [change tracking (aka delta query)](./deltas.md).

As an API owner you should implement both those patterns to enable new scenarios for your APIs users but also to save COGS (reducing the need for apps to perform continuous polling on your API).
