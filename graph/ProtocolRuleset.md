[[_TOC_]]

API owners that have onboarded to Microsoft Graph:

- [x] Preparing for the API review process
- [x] Designing new APIs or updating existing ones

and are looking to do one or more of the following:

- [ ] Understand the requirements of Microsoft Graph APIs
- [ ] Address issues raised by the schema validation CI pipeline
- [ ] Fix existing issues because grace period is expiring

# Schema validation

Currently schema validation is run by [Graph-Studio](Update-schema/Graph-Studio) on the build pipeline when workloads push to their test branches or create a pull request to master.
Workloads should have the prerogative to address all the errors raised by the validation and ensure that their schema is as compliant as possible
to the rules defined before publishing their changes.

There is however the ability to suppress **noncritical errors** for some time, to allow workloads to
plan and address errors that cannot be immediately resolved. Please see [Tracing-and-suppressions](Update-schema/Graph-Studio/Tracing-and-suppressions).

Error messages can have the following severity levels:

| Severity    | Description                                                                                            |
| :---------- | :----------------------------------------------------------------------------------------------------- |
| Critical    | Must be fixed before publishing. Cannot be suppressed. This Error is likely to break AGS if published. |
| Error       | Can be suppressed during publishing. This is a hard error that should be fixed before moving to v1.0.  |
| Warning     | Can be suppressed during publishing. This is a suggestion so as to conform to our coding style.        |
| Information | No need to suppress. It will not block publishing. This is for information purposes only.              |

# OData validation

Graph Studio performs the full suite of OData validations. Because Microsoft Graph rejects any schema containing OData violations, these violations are treated as `Critical` errors and cannot be suppressed. OData violation error codes are prefixed with `Schema.OData.{EdmErrorCode}`.

Example OData infraction:

```log
2020-11-30 21:47:51Z Critical Schema.OData.InvalidName: /Schemas/beta-Prod.csdl: [env=Prod;version=beta] '/ComplexType[testType]/Property[invalidProperty ]' The specified name is not allowed: 'invalidProperty '.
```

For a complete list of the OData error codes please see the [OData Validation Documentation](https://docs.microsoft.com/en-us/dotnet/api/microsoft.odata.edm.validation.edmerrorcode).

# Breaking change analysis

Breaking change analysis is performed by comparing latest (master) schema with current local schema. Each entry provides the file where the breaking change has occurred, and the element which triggered the error. The errors fall into one of three buckets:

- `Schema.BreakingChange.CannotAdd`
- `Schema.BreakingChange.CannotChange`
- `Schema.BreakingChange.CannotDelete`

Making changes to AGS annotations is not considered breaking, except adding `ags:IsHidden="true"` which makes an already public API private. Making changes to existing elements, or removing existing elements is considered breaking. Adding new elements is allowed and not considered breaking, but there are exceptions:

- Adding `EnumType` members for non-extensible enumerations is considered breaking.
- Adding `Nullable="false"` properties to existing types is considered breaking.
- Adding `Nullable="false"` parameters to existing actions and functions is considered breaking.
- Adding attributes to existing nodes is considered breaking.
  - Adding AGS annotations is exempted, except `ags:IsHidden="true"`.
  - Adding `OpenType="true"` is exempted.

# Validation for Private Preview API changes

Private preview API validation is performed by comparing latest (master) schema with current local schema. Each entry provides the file where the validation has occurred, and the element which triggered the error. The errors fall into one of the following buckets:

- `PrivatePreview.IsNotHidden`
- `PrivatePreview.Deprecated`
- `PrivatePreview.DeprecationDate`
- `PrivatePreview.RemovalDate`

The following are the rules that private preview API changes must follow:

- All elements added must be marked as hidden.
- All elements added must be deprecated.
- Deprecation date must be earlier than the current date.
- Removal date must not be later than 90 days from deprecation date.

# JSON Description Validation for Public Schema Changes

Pull requests with public schema changes need to have the below duly filled Json template as part of the pull request description: Please see [Pull-request-json-description](Update-schema/Pull-request-json-description).

If `IsAPIForPrivatePreview` is set to "Yes", then all changes need to have `ags:IsHidden="true"` and not appear in the final public metadata.
In this case, `ChangelogPullRequestUrl` and `DocumentationPullRequestUrl` can be left blank.
No change will go in without having `IsPrivacyReviewCompleted` set as yes and a valid `PrivacyReviewUrl`. Please see [Privacy-review](Privacy-review).

Errors from validation of the pull request description fall into one of the following buckets:

| Code                                                  | Severity   | Description                                                                           |
| :---------------------------------------------------- | :--------- | :------------------------------------------------------------------------------------ |
| `Schema.PullRequest.InvalidDescriptionJson`           | `Critical` | The pull request description Json is missing or has some formatting issues.           |
| `Schema.PullRequest.ValueMissingError`                | `Critical` | A required value is missing from the description Json.                                |
| `Schema.PullRequest.KeyMissingError`                  | `Critical` | A required key is missing from the description Json.                                  |
| `Schema.PullRequest.PrivacyReviewNotCompleted`        | `Critical` | The IsPrivacyReviewCompleted value must be set to True / Yes.                         |
| `Schema.PullRequest.InvalidPrivacyReviewUrl`          | `Critical` | The PrivacyReviewUrl is not a valid privacy review URL.                               |
| `Schema.PullRequest.InvalidGithubUrl`                 | `Critical` | The DocumentationPullRequestUrl / ChangelogPullRequestUrl must be a valid Github URL. |
| `Schema.PullRequest.InvalidAPIReviewUrl`              | `Critical` | The APIReviewApprovalPullRequestUrl is not a valid API review URL.                    |
| `Schema.PullRequest.PullRequestFetchError`            | `Critical` | There was an error fetching the API review pull request.                              |
| `Schema.PullRequest.APIReviewPullRequestNotCompleted` | `Critical` | The API review pull request is not complete.                                          |

# Microsoft Graph ruleset

Beyond OData and Breaking Change analysis, Graph Studio performs its own set of schema validations. These are typically best practices and naming conventions, but could also find semantic issues with the schemas. Before a schema can be published we must validate that it can be loaded by Microsoft Graph, which has its own set of criteria as to what makes a schema valid across all workloads. Because Microsoft Graph will reject any schemas not conforming to its criteria, any infractions are handled as `Critical` errors and cannot be suppressed.

| Code                                                           | Severity   | Description                                                                                                                                                                  |
| :------------------------------------------------------------- | :--------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Naming Validation**                                          |            |                                                                                                                                                                              |
| `Schema.Validation.CamelCase`                                  | `Error`    | Names must be in lower camel case.                                                                                                                                           |
| `Schema.Validation.NamespaceCamelCase`                         | `Error`    | Namespaces must be in lower camel case.                                                                                                                                      |
| `Schema.Validation.UseEmail`                                   | `Warning`  | A property name should use `email` instead of `mail`.                                                                                                                        |
| `Schema.Validation.SuffixTime`                                 | `Error`    | If a property has the type `Edm.Time`, its name must end in `Time` e.g. `startTime`.                                                                                         |
| `Schema.Validation.SuffixDate`                                 | `Error`    | If a property has the type `Edm.Date`, its name must end in `Date` (e.g. `birthDate`) or `MonthYear` (e.g. `startMonthYear`).                                                |
| `Schema.Validation.SuffixDateTime`                             | `Error`    | If a property has the type `DateTimeOffset`, its name must end in `DateTime` (e.g. `receivedDateTime`).                                                                      |
| *`Schema.Validation.Case2LetterAcronyms`                       | `Error`    | 2 letter acronyms should be cased with the same case (e.g. `prDescription`, `availableOnPC`).                                                                                |
| *`Schema.Validation.Case3PlusLetterAcronyms`                   | `Error`    | 3+ letter acronyms should be cased the same way as regular words (e.g. `adoPipeline`, `advancedRpc`).                                                                        |
| **Primary key validation**                                     |            |                                                                                                                                                                              |
| `Schema.Validation.EntityKeyMustBeString`                      | `Error`    | Check to verify that the primary id of an entity type is `string`.                                                                                                           |
| `Schema.Validation.PrimaryKeyMustBeDefinedAsProperty`          | `Error`    | Check to verify that the primary key must also be defined as a property.                                                                                                     |
| `Schema.Validation.PrimaryKeyMustNotBeComposite`               | `Error`    | Check to verify that the primary key is composed of a single property  and not multiple.                                                                                     |
| `Schema.Validation.AvoidComplexTypeId`                         | `Error`    | A complex type must not have the property `id`.                                                                                                                              |
| **Property name validation**                                   |            |                                                                                                                                                                              |
| `Schema.Validation.PropertyMustNotBeNamedType`                 | `Error`    | A property name should not be `"type"`.                                                                                                                                      |
| `Schema.Validation.PropertyNamesShouldNotStartWithTypeName`    | `Error`    | Property names should not start with type name.                                                                                                                              |
| `Schema.Validation.PropertyNameMustNotEndInPrimitiveType`      | `Error`    | Property names must not end in primitive types unless the type is temporal.                                                                                                  |
| `Schema.Validation.SingularNoun`                               | `Warning`  | Non-collection property names should be singular.                                                                                                                            |
| `Schema.Validation.PluralNoun`                                 | `Warning`  | Collection property names should be plural.                                                                                                                                  |
| `Schema.Validation.EntityTypeNameShouldBeSingular`             | `Warning`  | Entity type name should be singular.                                                                                                                                         |
| **Enum validation**                                            |            |                                                                                                                                                                              |
| `Schema.Validation.EnumShouldBeEvolvable`                      | `Warning`  | Enums should be evolvable.                                                                                                                                                   |
| `Schema.Validation.EnumMemberValuesShouldBeZeroOrPowersOfTwo`  | `Warning`  | Consider using zero or powers of two for flag enum member values.                                                                                                            |
| **Stream validation**                                          |            |                                                                                                                                                                              |
| `Schema.Validation.MediaEntityTypesCannotContainSubstreams`    | `Warning`  | Streams must not define a property of type `Edm.Stream`.                                                                                                                     |
| `Schema.Validation.MediaEntityTypesCannotInheritFromABaseType` | `Warning`  | Streams cannot inherit from a base type.                                                                                                                                     |
| **Structure validation**                                       |            |                                                                                                                                                                              |
| `Schema.Validation.OperationsMustBeBound`                      | `Error`    | Actions and Functions must have an IsBound='true' attribute and the first parameter must be the binding parameter. This is an AGS limitation.                                |
| `Schema.Validation.NavigationPropertyBindingMissing`           | `Warning`  | `NavigationProperty` without `ContainsTarget` must define `NavigationPropertyBinding` in Singleton/EntitySet.                                                                |
| `Schema.Validation.OperationsShouldBeAvoided`                  | `Warning`  | Operations with names containing add, create, update, delete or remove should be avoided whenever possible.                                                                  |
| *`Schema.Validation.ParallelCollections`                       | `Error`    | Do not use parallel collections; use collections of complex types instead.                                                                                                   |
| *`Schema.Validation.ProperCollections`                         | `Warning`  | Consider using a proper collection rather than `property1`, `property2`, etc.                                                                                                |
| *`Schema.Validation.EntitySetNavigationProperties`             | `Error`    | Entity sets should have valid navigation properties.                                                                                                                         |
| *`Schema.Validation.SingletonNavigationProperties`             | `Error`    | Singletons should have valid navigation properties.                                                                                                                          |
| **Cross schema validation**                                    |            |                                                                                                                                                                              |
| `Schema.Validation.EntityWithoutMaster`                        | `Critical` | Ensure that all entities have the ags:IsMaster="true" or ags:IsShared="true" annotation.                                                                                     |
| `Schema.Validation.TypeOverridesBaseProperty`                  | `Critical` | Types cannot override their base properties.                                                                                                                                 |
| `Schema.Validation.PropertyAlreadyExists`                      | `Critical` | Entity type cannot redefine properties already defined by another workload.                                                                                                  |
| `Schema.Validation.InconsistentSharedType`                     | `Critical` | Ensure that shared type definitions are consistent across workloads.<br />If multiple workloads define a shared type, they must have the exact same definition of that type. |
| `Schema.Validation.ElementAlreadyExists`                       | `Critical` | Different elements cannot share the same name.<br />If you have an entity called `foo`, you cannot have an action `foo` in the same namespace.                               |
| `Schema.Validation.NavigationPropertyContainsForeignTarget`    | `Error`    | Navigation property cannot contain target from a different workload.                                                                                                         |

\*Validation has not yet been automated in Graph Studio. However, workloads should adhere to these specifications so as not to risk having their schemas broken in the future.

# Microsoft Graph protocol ruleset (Not implemented yet)

| Name                                                                                            | Severity |
| ----------------------------------------------------------------------------------------------- | -------- |
| **Request patterns**                                                                            |          |
| ✔ DO use `GET …/{collection}` and `GET …/{collection}/{id}` for listing and reading resources.  | Error    |
| ✔ DO use `POST …/{collection}` for creating resources.                                          | Error    |
| ✔ DO use `PATCH …/{collection}/{id}` for updating resources.                                    | Error    |
| ✖ AVOID using `PUT …/{collection}/{id}` for updating resources.                                 | Warning  |
| ✖ DO NOT use `PATCH` to replaces resources or `PUT` to partially update resources.              | Error    |
| ✖ AVOID patterns that require multiple round trips to complete a single logical action.         | Warning  |
| ✔ CONSIDER supporting `return`, and `omit-nulls` preferences.                                   | Warning  |
| **Serialization**                                                                               |          |
| ✔ DO use an object as the root of all JSON payloads.                                            | Error    |
| ✔ DO use a `value` property in the root object to return a collection.                          | Error    |
| ✔ DO include `@odata.type` annotations when the type is ambiguous.                              | Warning  |
| **Authorization**                                                                               |          |
| ✖ DO NOT use a scope ending with `.Read` to authorize a data modification.                      | Error    |
| **Errors**                                                                                      |          |
| ✔ DO return an `error` property with a child `code` property in all error responses.            | Error    |
| ✔ DO return a `403 Forbidden` error when insufficient scopes are present on the auth token.     | Error    |
| ✔ CONSIDER returning a `404 Not found` error if a `403` would result in information disclosure. | Warning  |
| ✔ DO return a `429 Too many requests` error when the caller has exceeded throttling limits.     | Error    |

# Contacts

| Area    | Contact                                                                                           |
| :------ | :------------------------------------------------------------------------------------------------ |
| Support | [StackOverflow](https://stackoverflow.microsoft.com/questions/tagged/1096) tag `[MicrosoftGraph]` |
