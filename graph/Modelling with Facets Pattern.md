# Facets Pattern

Microsoft Graph API Design Pattern

### *A frequent pattern in Microsoft Graph is to model multiple variants of a common concept as a single entity type with common properties and facets for variants.*


## Problem
API developer needs to model a set of heterogeneous resources that have common properties and behaviors, and may express features of multiple variants at a time.
For example a movie clip stored on OneDrive has properties of File Type and Video Type.

## Solution

API designers can create multiple complex types to bundle properties for each variant then define a parent entity type with common properties and one property of a complex type per variant.
In this solution a child variant is identified by a presence of one or multiple facets in the parent object.

## Issues and Considerations

When introducing a new subtype, you need to ensure that the new subtype doesn't
change the semantic of the type hierarchy with it's implicit constraints. For example 

When introducing a new subtype to the hierarchy, developers need to ensure that
the new subtype doesn't change the semantic of the type hierarchy with its
implicit constraints.

There are a **few potential risks** for client applications when new sub-types
are introduced:

-   De-serialization code might break because of missing properties in returned
    collection items. Even though property X was mandatory on all subtypes
    previously returned, the new subtype might not have this property and the
    client code needs to deal with that.

-   Client libraries for strongly typed language might ignore some of the values
    in the @odata.type property without further configuration and need to be
    updated to be able to pick the right (client) type to deserialize into.

In addition, you can follow some of the mitigation techniques such as:

-   Think about gradual roll-out sequence

    -   Consider that Microsoft Graph does not return objects from a workload
        that has a type that is not configured in current metadata. To avoid
        inconsistencies, follow a two-step process:

        -   Introduce the entity type to the Graph metadata but don’t return
            objects of the type in any of the heterogeneous collections.

        -   Enable your workload to return objects of the new type as items of
            collection.

-   Allow time for testing

    -   Inform the clients about the change and allow them to test the changes
        in beta. Time is required to implement the code necessary to deal with
        the new entity type, both in terms of de-serialization as well as
        integrating it into the rest of the application.

-   Communicate the change in semantics

    -   It is necessary for the client developers to incorporate the new
        semantic into their application/service, even if the change is perceived
        to be small. This requires early communication and clear documentation
        of what the new type represents and why/how it is considered a subtype
        of the original abstract type of the collection.

## When to Use this Pattern

The facet pattern is useful when ... 
 make it easier to query resources using OData $filter expression
This pattern is useful with relatively small number of subtypes otherwise the main object become very sparsely populated.

There are related patterns to consider such as
[Type Hierarchy](https://github.com/microsoft/api-guidelines/tree/graph/graph) and [Flat
bag of
properties](https://github.com/microsoft/api-guidelines/tree/graph/graph).

## Example
```XML
  <EntityType Name="driveItem" BaseType="graph.baseItem" OpenType="true" ags:MasterService="Microsoft.FileServices" ags:WorkloadIds="Microsoft.Excel,Microsoft.Powerpoint,Microsoft.Teams.GraphSvc,Microsoft.Word">
        <Property Name="audio" Type="graph.audio" />
        <Property Name="bundle" Type="graph.bundle" />
        <Property Name="content" Type="Edm.Stream" />
        <Property Name="cTag" Type="Edm.String" />
        <Property Name="deleted" Type="graph.deleted" />
        <Property Name="file" Type="graph.file" />
        <Property Name="fileSystemInfo" Type="graph.fileSystemInfo" />
        <Property Name="folder" Type="graph.folder" />
        <Property Name="image" Type="graph.image" />
        <Property Name="location" Type="graph.geoCoordinates" />
        <Property Name="malware" Type="graph.malware" />
        <Property Name="media" Type="graph.media" />
        <Property Name="package" Type="graph.package" />
        <Property Name="pendingOperations" Type="graph.pendingOperations" />
        <Property Name="photo" Type="graph.photo" />
        <Property Name="publication" Type="graph.publicationFacet" />
        <Property Name="remoteItem" Type="graph.remoteItem" />
        <Property Name="root" Type="graph.root" />
        <Property Name="searchResult" Type="graph.searchResult" />
        <Property Name="shared" Type="graph.shared" />
        <Property Name="sharepointIds" Type="graph.sharepointIds" />
        <Property Name="size" Type="Edm.Int64" />
        <Property Name="source" Type="graph.driveItemSource" />
        <Property Name="specialFolder" Type="graph.specialFolder" />
        <Property Name="video" Type="graph.video" />
        <Property Name="webDavUrl" Type="Edm.String" />
     ...
      </EntityType>
```



```JSON
https://graph.microsoft.com/beta/me/drive/root/children

Response shortened for readability:
  {
            "@microsoft.graph.downloadUrl": "https://microsoft-my.sharepoint-df.com/personal/opodolyako_microsoft_com/_layouts/15/download.aspx?UniqueId=66428197-fb89-4611-91f7-77e6d03245b4&Translate=false&tempauth=eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.J1c2VQZXJzaXN0ZW50Q29va2llIjpudWxsLCJpcGFkZHIiOiIyMC4xOTAuMTM1LjQzIn0.SU9ZM2FCa2xaM2UyaC85d0hUNmN4bmU2cEJDZGdncEdtQ0FmM0llR0tUbz0&ApiVersion=2.0",
            "createdDateTime": "2021-12-15T00:07:36Z",
            "eTag": "\"{66428197-FB89-4611-91F7-77E6D03245B4},2\"",
            "id": "01XXNRXFEXQFBGNCP3CFDJD53X43IDERNU",
            "lastModifiedDateTime": "2021-12-15T00:07:36Z",
            "name": "Versioning and Deprecation.docx",
            "webUrl": "https://microsoft-my.sharepoint-df.com/personal/opodolyako_microsoft_com/_layouts/15/Doc.aspx?sourcedoc=%7B66428197-FB89-4611-91F7-77E6D03245B4%7D&file=Versioning%20and%20Deprecation.docx&action=default&mobileredirect=true",
            "cTag": "\"c:{66428197-FB89-4611-91F7-77E6D03245B4},1\"",
            "size": 21400,
           ...           
            "file": {
                "mimeType": "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
                "hashes": {
                    "quickXorHash": "r2d9uZilW0zEIXwycymsUQzhV+U="
                }
            },
            "fileSystemInfo": {
                "createdDateTime": "2021-12-15T00:07:36Z",
                "lastModifiedDateTime": "2021-12-15T00:07:36Z"
            }
        },
        {
            "@microsoft.graph.downloadUrl": "https://microsoft-my.sharepoint-df.com/personal/opodolyako_microsoft_com/_layouts/15/download.aspx?UniqueId=d13e47ea-a6db-4d30-affd-e82dbf01cb51&Translate=false&tempauth=eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.J1c2VQZXJzaXN0ZW50Q29va2llIjpudWxsLCJpcGFkZHIiOiIyMC4xOTAuMTM1LjQzIn0.TWdMOUhoNDlvSEN5UHM5S3VoMms3Nk9IRldPVWJzSDBlb0xRV3Vld244bz0&ApiVersion=2.0",
            "createdDateTime": "2021-12-21T16:32:51Z",
            "eTag": "\"{D13E47EA-A6DB-4D30-AFFD-E82DBF01CB51},1\"",
            "id": "01XXNRXFHKI47NDW5GGBG277PIFW7QDS2R",
            "lastModifiedDateTime": "2021-12-21T16:32:51Z",
            "name": "WhaleShark.jpg",
            "webUrl": "https://microsoft-my.sharepoint-df.com/personal/opodolyako_microsoft_com/Documents/WhaleShark.jpg",
            "cTag": "\"c:{D13E47EA-A6DB-4D30-AFFD-E82DBF01CB51},1\"",
            "size": 29097,
            .....
            "file": {
                "mimeType": "image/jpeg",
                "hashes": {
                    "quickXorHash": "2vHpAA7RDZJteIwl1pXR980xuh4="
                }
            },
            "fileSystemInfo": {
                "createdDateTime": "2021-12-21T16:32:51Z",
                "lastModifiedDateTime": "2021-12-21T16:32:51Z"
            },
            "image": {}
        },
```
