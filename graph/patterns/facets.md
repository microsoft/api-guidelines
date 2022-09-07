# Facets

Microsoft Graph API Design Pattern

*A frequent pattern in Microsoft Graph is to model multiple variants of a common concept as a single entity type with common properties and facets for variants.*

## Problem

An API designer needs to model a set of heterogeneous resources that have common properties and behaviors and might express features of multiple variants at a time because variants are not mutually exclusive.
For example, a movie clip stored on OneDrive is both a file and a video. There are properties associated to each variant.

## Solution

API designers create multiple complex types to bundle properties for each variant, and then define an entity type with a property for each complex type to hold the properties of the variant.

In this solution, a child variant is identified by the presence of one or more facets in the parent object.

## When to use this pattern

The facets pattern is useful when there is a number of variants and they are not mutually exclusive. It also makes it syntactically easier to query resources by using the OData `$filter` expression because it doesn't require casting.

You can consider related patterns such as [type hierarchy](./subtypes.md) and [flat bag of properties](./flat-bag.md).

## Issues and considerations

When introducing a new facet, you need to ensure that the new facet doesn't change the semantic of the model with its implicit constraints.

## Example

The driveItem resource represents a file, folder, image, or other item stored in a drive and is modeled by using an entity type with multiple facets.

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

An API request to get all items from a personal OneDrive returns a heterogenous collection with different facets populated. In the following example, there is a folder, a file, and an image in the collection. The image entity has two facets populated: file and image.

```
GET https://graph.microsoft.com/v1.0/me/drive/root/children

Response shortened for readability:
 
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users('93816c1c-1b19-41de-a322-a1643d7f4d39')/drive/root/children",
    "value": [
        {
            "createdDateTime": "2021-07-07T13:59:47Z",
            "name": "Microsoft Teams Chat Files",
             ...,
            "folder": {
                "childCount": 15
            }
        },
        ...
       {
            "createdDateTime": "2021-12-15T00:07:36Z",
            "name": "Versioning and Deprecation.docx",          
            ...,           
            "file": {
                "mimeType": "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
                "hashes": {
                    "quickXorHash": "r2d9uZilW0zEIXwycymsUQzhV+U="
                }
            },
           ...
        },
        {
            "createdDateTime": "2021-12-21T16:32:51Z",
            "name": "WhaleShark.jpg",
            ...
            "file": {
                "mimeType": "image/jpeg",
                "hashes": {
                    "quickXorHash": "2vHpAA7RDZJteIwl1pXR980xuh4="
                }
            },
           ...,
            "image": {}
        }
    ]
```
