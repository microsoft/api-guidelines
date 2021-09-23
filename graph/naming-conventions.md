
### General Guidelines

::: tip ✔ DO use `lowerCamelCase` for _all_ names.

- Right: `automaticRepliesStatus`.
- Wrong: `kebab-case` or `snake_case`.

:::

::: warning ✖ AVOID redundant words in names.

- Right: `/places/{id}/`**_type_** and `/phones/{id}/`**_number_**
- Wrong: `/places/{id}/`_**placeType**_ and `/phones/{id}/`**_phoneNumber_**

:::

::: warning ✖ AVOID using brand names in type or property names.

- Right: `chat`
- Wrong: `teamsChat`

:::

::: warning ✖ AVOID using acronyms or abbreviations unless they are broadly understood.

- Right: `url` or `htmlSignature`
- Wrong: `msodsUrl` or `dlp`

:::

::: tip ✔ DO use singular nouns for type names.

- Right: `address`
- Wrong: `addresses`

:::

::: tip ✔ DO use plural nouns for collections (for listing a type or collection properties).

- Right: `addresses`
- Wrong: `address`

:::

::: tip ✔ DO pluralize the noun even when followed by an adjective (a "postpositive").

- Right: `passersby` or `mothersInLaw`
- Wrong: `notaryPublics` or `motherInLaws`

:::

### Casing

::: tip ✔ DO case two-letter acronyms with the same case.

- Right: `ioLimit` or `totalIOAmount`
- Wrong: `iOLimit` or `totalIoAmount`

:::

::: tip ✔ DO case three+ letter acronyms the same as a normal word.

- Right: `fidoKey` or `oauthUrl`
- Wrong: `webHTML`

:::

::: danger ✖ DO NOT capitalize the word following a <a href="https://www.thoughtco.com/common-prefixes-in-english-1692724">prefix</a> or words within a <a href="http://www.learningdifferences.com/Main%20Page/Topics/Compound%20Word%20Lists/Compound_Word_%20Lists_complete.htm">compound word</a>.

- Right: `subcategory`, `geocoordinate` or `crosswalk`
- Wrong: `metaData`, `semiCircle` or `airPlane`

:::

::: tip ✔ DO capitalize within hyphenated and open (spaced) compound words.

- Right: `fiveYearOld`, `daughterInLaw` or `postOffice`
- Wrong: `paperclip`, `changingroom` or `fullmoon`

:::

### Prefixes and Suffixes

::: tip ✔ DO suffix date and time properties.

- Right: `dueDate`&mdash;an `Edm.Date`
- Right: `createdDateTime`&mdash;an `Edm.DateTimeOffset`
- Right: `recurringMeetingTime`&mdash;an `Edm.TimeOfDay`
- Wrong: `dueOn` or `startTime`, both an `Edm.DateTimeOffset`

:::

::: danger ✖ DO NOT suffix property names with primitive type names unless the type is temporal.

- Right: `isEnabled` or `amount`
- Wrong: `enabledBool`

:::

::: tip ✔ DO prefix property names for properties concerning a different entity.

- Right: `siteWebUrl` on `driveItem`, or `userId` on `auditActor`
- Wrong: `webUrl` on `contact` when its the `companyWebUrl`

:::

### Common property names

| Approved name          | Type           | Use                                                       |
| ---------------------- | -------------- | --------------------------------------------------------- |
| `displayName`          | String         | A label that can be displayed or read aloud. Not `name`.  |
| `webUrl`               | String         | The web page for viewing or editing this entity.          |
| `url`                  | String         | A URL to a resource. (In Graph often holds the `webUrl`.) |
| `lastModifiedDateTime` | DateTimeOffset | The last time this entity changed.                        |
| `createdDateTime`      | DateTimeOffset | The time this entity was created.                         |
| `createdBy`            | identitySet    | The creator of this entity.                               |
| `createdByUser`        | user           | The user in `/users` who created this entity.             |

