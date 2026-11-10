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

## Facets in TypeSpec

In [TypeSpec](https://aka.ms/typespec), the facets pattern is expressed with the `@facet` decorator from the `@microsoft/typespec-msgraph` library. A facet is a **nullable property** on an `@entity` model whose type is a `@complex` model. The `@facet` decorator marks the property as a variant facet so that authoring tooling and linters can validate it.

The following TypeSpec models the same `driveItem` facets shown in the CSDL example above — each variant (`audio`, `file`, `folder`, `image`, `video`) is a complex type, and the entity carries one nullable facet property per variant:

```typespec
// One @complex type per variant facet
@complex
model audio {
  @doc("The title of the album for this audio file.")
  album: string | null;
}

@complex
model file {
  @doc("The MIME type for the file.")
  mimeType: string | null;
}

@complex
model folder {
  @doc("Number of children contained immediately within this container.")
  childCount: int32 | null;
}

@complex
model image {
  @doc("Width of the image, in pixels.")
  width: int32 | null;

  @doc("Height of the image, in pixels.")
  height: int32 | null;
}

@complex
model video {
  @doc("Duration of the video, in milliseconds.")
  duration: int64 | null;
}

// The entity declares one nullable facet property per variant
@entity
model driveItem {
  @key id: string;

  @doc("The name of the item (file name, folder name).")
  displayName: string | null;

  @doc("Audio facet. Present when the item is an audio file.")
  @facet audio: audio | null;

  @doc("File facet. Present when the item is a file.")
  @facet file: file | null;

  @doc("Folder facet. Present when the item is a folder.")
  @facet folder: folder | null;

  @doc("Image facet. Present when the item is an image.")
  @facet image: image | null;

  @doc("Video facet. Present when the item is a video.")
  @facet video: video | null;
}
```

### Rules for `@facet`

The `@facet` decorator is validated by the `@microsoft/typespec-msgraph` linter:

- The property type must be a `@complex` model — primitive, enum, and entity types are rejected.
- The property must be **nullable** (`T | null`) — a variant may be absent.
- The property must be declared on an `@entity` model — facets on `@complex` models are rejected.

### Compiled CSDL

`@facet` is an **authoring-time marker**: it carries no wire-format meaning, so a facet property compiles to a standard nullable complex-typed property — identical to what a hand-authored facet produces. The TypeSpec above emits:

```xml
<EntityType Name="driveItem">
  <Key>
    <PropertyRef Name="id"/>
  </Key>
  <Property Name="id" Type="Edm.String" Nullable="false"/>
  <Property Name="displayName" Type="Edm.String" Nullable="true"/>
  <Property Name="audio" Type="graph.audio" Nullable="true"/>
  <Property Name="file" Type="graph.file" Nullable="true"/>
  <Property Name="folder" Type="graph.folder" Nullable="true"/>
  <Property Name="image" Type="graph.image" Nullable="true"/>
  <Property Name="video" Type="graph.video" Nullable="true"/>
</EntityType>
```

Because `@facet` adds no CSDL annotation, applying or omitting it produces byte-identical metadata; its value is the design intent it records and the linter checks it enables.
