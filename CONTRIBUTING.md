# Contributing to the Microsoft REST API Guidelines
The Microsoft REST API Guidelines is a Microsoft-wide initiative to develop consistent design guidelines for REST APIs. The initiative requires input and feedback from a variety of individuals both inside and outside of Microsoft.

To provide feedback, please follow the guidance in this document. Please note that these are just guidelines, not rules. Use your best judgment and feel free to propose changes to anything in this repository, including the contribution guidelines.

Please note that this project is released with a [Contributor Code of Conduct][code-of-conduct]. By participating in this project you agree to abide by its terms.
- [Creating issues](#creating-issues)
- [Recommended setup for contributing](#recommended-setup-for-contributing)
- [Documentation styleguide](#documentation-styleguide)
- [Commit messages](#commit-messages)
- [Pull requests](#pull-requests)

## Creating issues
- You can [create an issue][new-issue], but before doing that please read the bullets below and include as many details as possible.
- Perform a [cursory search][issue-search] to see if a similar issue has already been submitted.
- Reference the version of the Microsoft REST API Guidelines you are using.
- Include the guidance you expected and other places you've seen that guidance, e.g. [White House Web API Standards][white-house-api-guidelines].
- Include sample requests and responses whenever possible.

### Related repositories
This is the repository for Microsoft REST API Guidelines documentation only. Please ensure that you are opening issues in the right repository.

## Recommended setup for contributing
- Fork this repository on GitHub
- Install [Git][git] to your computer and clone your new forked repository
- Install [Atom][atom], [VS Code][vscode], or your favorite editor
- Install [markdown-toc package][markdown-toc]

## Documentation styleguide
- Use [GitHub-flavored markdown][gfm]
- Use syntax-highlighted examples liberally
- Trim trailing empty lines from HTTP requests
- Retain only essential headers for understanding the example
- Use valid (e.g., member names quoted), pretty-printed JSON with a 2 space indent
- Minimize JSON payloads by using ellipses
- Write one sentence per line.

### Example
#### Request

```http
GET http://services.odata.org/V4/TripPinServiceRW/People HTTP/1.1
Accept: application/json
```

#### Response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "@nextLink":"http://services.odata.org/V4/TripPinServiceRW/People?$skiptoken=8",
  "value":[
    {
      "userName":"russellwhyte",
      "firstName":"Russell",
      "lastName":"Whyte",
      "emails":[
        "Russell@example.com",
        "Russell@contoso.com"
      ],
      "addressInfo":[
        {
          "address":"187 Suffolk Ln.",
          "city":{
            "countryRegion":"United States",
            "name":"Boise",
            "region":"ID"
          }
        }
      ],
      "gender":"Male",
    },
    ...
  ]
}
```

## Commit messages
- Use the present tense: "Change ...", not "Changed ..."
- Use the imperative mood: "Change ...", not "Changes ..."
- Limit the first line to 72 characters or less
- Reference issues and pull requests liberally

## Pull requests
Pull requests serve as the primary mechanism by which contributions are proposed and accepted. We recommend creating a [topic branch][topic-branch] and sending a pull request to the `vNext` branch from the topic branch. For additional guidance, read through the [GitHub Flow Guide][github-flow-guide].

Be prepared to address feedback on your pull request and iterate if necessary.

[code-of-conduct]: https://opensource.microsoft.com/codeofconduct/
[new-issue]: https://github.com/Microsoft/api-guidelines/issues/new
[issue-search]: https://github.com/Microsoft/api-guidelines/issues
[white-house-api-guidelines]: https://github.com/WhiteHouse/api-standards/blob/master/README.md
[topic-branch]: http://www.git-scm.com/book/en/v2/Git-Branching-Branching-Workflows#Topic-Branches
[gfm]: https://guides.github.com/features/mastering-markdown/#GitHub-flavored-markdown
[github-flow-guide]: https://guides.github.com/introduction/flow/
[atom-beautify]: https://atom.io/packages/atom-beautify
[atom]: http://atom.io
[markdown-toc]: https://atom.io/packages/markdown-toc
[vscode]: https://code.visualstudio.com/
[git]: https://git-scm.com/
