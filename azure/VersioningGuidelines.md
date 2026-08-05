# Azure Versioning Guidelines

## History

<details>
  <summary>Expand change history</summary>

| Date        | Notes                                                          |
| ----------- | -------------------------------------------------------------- |
| 2024-Nov-14 | Azure Service Versioning & Breaking Change Guidelines       |

</details>

## Guidelines

This document provides a "Dos and Don'ts" list for complying with the Azure Versioning and Breaking Change Policy,
as documented [internally](aka.ms/AzBreakingChangesPolicy) and [externally](https://learn.microsoft.com/azure/developer/intro/azure-service-sdk-tool-versioning).

:white_check_mark: **DO** thoroughly ensure/test the API contract is entirely correct before merging it into a production branch of the specs repo.

Testing helps avoid "BugFix" changes to the API definition. Testing should be done at the HTTP level as well as through generated SDKs.

:white_check_mark: **DO** retire all prior preview API versions 90 days after a new GA or preview API version is released.

:white_check_mark: **DO** contact the Azure Breaking Change Review board to coordinate communications to customers
when releasing an API version requiring the retirement of a prior version.

:white_check_mark: **DO** create a new preview API version for any features that should remain in preview following a new GA release.

:white_check_mark: **DO** use a date strictly later than the most recent GA API version when releasing
a new preview API version.

:white_check_mark: **DO** deprovision any API version that has been retired. Retired APIs versions should behave like
an unknown API version (see [ref](https://aka.ms/azapi/guidelines#versioning-api-version-unsupported)).

:white_check_mark: **DO** remove retired API versions from the azure-rest-api-specs repo.

:white_check_mark: **DO** review any change to service behavior that could disrupt customers with the Azure Breaking Changes review board, even if the change is not part of the API definition.

Some examples of behavior changes that must be reviewed are:
- Introducing or changing rate limits to be more restrictive than previously
- Changing the permissions required to successfully execute an operation

:no_entry: **DO NOT** change the behavior of an API version that is available to customers either in public preview or GA.
Changes in behavior should always be introduced in a new API version, with prior versions working as before.

:no_entry: **DO NOT** introduce breaking changes from a prior GA version just to satisfy ARM or Azure API guidelines.

Avoiding breaking changes in a GA API takes precedence over adherence to API guidelines and resolving linter errors.

:no_entry: **DO NOT** keep a preview feature in preview for more than 1 year; it must go GA (or be removed) within 1 year after introduction.
