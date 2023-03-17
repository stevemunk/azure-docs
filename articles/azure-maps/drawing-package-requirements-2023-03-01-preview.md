---
title: Drawing package requirements in Microsoft Azure Maps Creator
titleSuffix: Microsoft Azure Maps Creator
description: Learn about the drawing package requirements to convert your facility design files to map data in the 2023-03-01-preview release
author: brendansco
ms.author: Brendanc
ms.date: 03/31/2023
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps

---

# Drawing package requirements

You can convert uploaded drawing packages into map data by using the Azure Maps [Conversion service]. This article describes the drawing package requirements for the Conversion API. To view a sample package, you can download the [drawing package].

For a guide on how to prepare your drawing package, see [Conversion Drawing Package Guide].

## Changes and Revisions

- Added support for user defined feature classes.  
- Simplified requirements of DWG layers.  

## Prerequisites

The drawing package includes drawings saved in DWG format, which is the native file format for Autodesk's AutoCAD® software.

You can choose any CAD software to produce the drawings in the drawing package.

The [Conversion service] converts the drawing package into map data. The Conversion service works with the AutoCAD DWG file format AC1032.

## Glossary of terms

For easy reference, here are some terms and definitions that are important as you read this article.

| Term  | Definition                                                                                    |
|:------|:----------------------------------------------------------------------------------------------|
| Layer | An AutoCAD DWG layer from the drawing file.                                                   |
| Entity| An AutoCAD DWG entity from the drawing file.                                                  |
| Xref  | A file in AutoCAD DWG file format, attached to the primary drawing as an external reference.  |
| Level | An area of a facility at a set elevation. For example, the floor of a facility.               |
|Feature| An instance of an object produced from the Conversion service that combines a geometry with metadata information. |
|Feature classes| A common blueprint for features.                                                      |

## Drawing package structure

A drawing package is a ZIP archive that contains the following files:

- DWG files in AutoCAD DWG file format.
- A *manifest.json* file that describes the DWG files in the drawing package.

The drawing package must be compressed into a single archive file, with the .zip extension. The DWG files can be organized in any way inside the drawing package, but the manifest file must be in the root directory. The next sections explain the conversion process and requirements for both the DWG and manifest files, and the content of these files. To view a sample package, you can download the [sample drawing package].

## DWG file conversion process

The Azure Maps Conversion service converts DWG file(s) of a facility to map data representing a facility and features of a facility.

The Azure Maps Conversion service creates:

- **Facility Feature**: The top-level feature of a facility that all levels of a facility are associated to. This is defined as the **Exterior** property in the [Georeference] tab of the onboarding tool.
- **Level features**: One Level feature is created for each floor of a facility. All features on a level are associated with a level.
- **User defined features**: DWG layers are mapped to a user defined [feature class](#featureclass) and become instances of the feature class.
- <!--DWG layers are mapped to a user defined [feature class](#featureclass). All mapped DWG layer entities become instances of the feature class.-->

## DWG file requirements

Each DWG file must adhere to these requirements:

- The DWG file can't contain features from multiple facilities.
- The DWG file can't contain features from multiple levels. For example, a facility with three levels has three DWG files in the drawing package.
- All data of a single level must be contained in a single DWG file. Any external references (*xrefs*) must be bound to the parent drawing.
- The DWG file must define layer(s) representing the boundary of that level.
- The DWG must reference the same measurement system and unit of measurement as other DWG files in the drawing package.
- The DWG file must be aligned when stacked on top of another level from the same facility.

## DWG layer requirements

### Feature classes

One or more DWG layer(s) can be mapped to a user defined feature class. One instance of the feature is created from an entity on the mapped layer. For example, DWG layers chair, table, and couch are mapped to a feature class called furniture. A furniture feature is created for every entity from the defined layers. Additionally:

- All layers should be separated to represent different feature types of the facility.
- All entities must fall inside the bounds of the level perimeter.
- The following AutoCAD entity types are supported: asdf, asdf, asdf, asdf

### Feature class properties

Text entities that fall within the bounds of a closed shape can be associated to that feature as a property. For example, a room feature class might have text that describes the room name and another the room type [sample drawing package]. Additionally:

- Only TEXT and MTEXT entities will be associated to the feature as a property. All other entity types will be ignored.
- TEXT and MTEXT justification point must fall within the bounds of the closed shape.
- If more than one TEXT property is within the bounds of the closed shape and both are mapped to one property, one will randomly be selected.

### Facility level

The DWG file for each level must contain a layer to define that level's perimeter. For example, if a facility contains two levels, then it needs to have two DWG files, each with a layer that defines that level's perimeter.

No matter how many entity drawings are in the level perimeter layer, the resulting facility dataset contains only one level feature for each DWG file. Additionally:

- Level perimeters must be drawn as Polygon, Polyline (closed), Circle, or Ellipse (closed).
- Level perimeters may overlap but are dissolved into one geometry.
- The resulting level feature must be at least 4 square meters.
- The resulting level feature must not be greater than 400,000 square meters.

If the layer contains multiple overlapping Polylines, the Polylines are dissolved into a single Level feature. Instead, if the layer contains
multiple nonoverlapping Polylines, the resulting Level feature has a multi-polygonal representation.

You can see an example of the Level perimeter layer as the 'GROS$' layer in the [sample drawing package].

## Manifest file requirements

The drawing package must contain a manifest file at the root level and the file must be named **manifest.json**. It describes the DWG files
allowing the  [Conversion service] to parse their content. Only the files identified by the manifest are used. Files that are in the drawing package, but aren't properly listed in the manifest, are ignored.

The file paths in the buildingLevels object of the manifest file must be relative to the root of the drawing package. The DWG file name must exactly match the name of the facility level. For example, a DWG file for the "Basement" level is *Basement.dwg*. A DWG file for level 2 is named as *level_2.dwg*. Filenames can't contain spaces, you can use an underscore to replace any spaces.

Although there are requirements when you use the manifest objects, not all objects are required. The following table shows the required and optional objects for the 2023-03-01-preview [Conversion service].

> [!NOTE]
> Unless otherwise specified, all string properties are limited to one thousand characters.

### Manifest JSON file

| Property       | Type                          | Required | Description                                                                             |
|----------------|-------------------------------|----------|-----------------------------------------------------------------------------------------|
| `version`      | number                        | TRUE     | Manifest schema version. Currently version 2.0                                          |
|`buildingLevels`| [BuildingLevels] object       | TRUE     | Specifies the levels of the facility and the files containing the design of the levels. |
|`featureClasses`|Array of [featureClass] objects| TRUE     | List of feature class objects that define how layers are read from the DWG drawing file.|
| `georeference` |[Georeference](#georeference) object| FALSE | Contains numerical geographic information for the facility drawing.                   |
| `facilityName` | string                        | FALSE    | The name of the facility.                                                               |

The next sections detail the requirements for each object.

#### buildingLevels

| Property  | Type                   | Required | Description                                             |
|-----------|------------------------|----------|---------------------------------------------------------|
|`dwgLayers`| Array of strings | TRUE | Names of layers that define the exterior profile of the facility. |
| `levels`  | Array of level objects | TRUE | A level refers to a unique floor in the facility defined in a DWG file, the height of each level and vertical order in which they appear. |

#### level

| Property       | Type    | Required | Description                                                                              |
|----------------|---------|----------|------------------------------------------------------------------------------------------|
| `levelName`    | string  | TRUE  | The name of the level. For example: Floor 1, Lobby, Blue Parking, or Basement.              |
| `ordinal`      | integer | TRUE  | Defines the vertical order of levels. All `ordinal` values must be unique within a facility.|
| `filename`     | string  | TRUE  | The path and name of the DWG file representing the level in a facility. The path must be relative to the root of the drawing package.  |
|`verticalExtent`| number  | FALSE | Floor-to-ceiling vertical height (thickness) of the level in meters.                        |

#### featureClass

| Property               | Type                          | Required | Description                                 |
|------------------------|-------------------------------|----------|---------------------------------------------|
| `dwgLayers`| Array of strings| TRUE| The name of each layer that defines the feature class. Each entity on the specified layer is converted to an instance of the feature class. The `dwgLayer` name that a feature is converted from ends up as a property of that feature. |
| `featureClassName`     | String                        | TRUE     | The name of the feature class. Typical examples include room, workspace or wall.|
|`featureClassProperties`| Array of [featureClassProperty] objects | TRUE | Specifies text layers in the DWG file associated to the feature as a property. For example, a label that falls inside the bounds of a space, such as a room number.|

#### featureClassProperty

| Property     | Type      | Required | Description                 |
|--------------|-----------|----------|-----------------------------|
| `dwgLayers` | Array of strings | TRUE | The name of each layer that defines the feature class property. Each entity on the specified layer is converted to a property. Only the DWG `TEXT` and `MTEXT` entities are converted to properties. All other entities are ignored.  |
|`featureClassPropertyName`| String | TRUE | Name of the feature class property, for example, spaceName or spaceUseType.|

#### georeference

| Property | Type   | Required | Description                                                                                                                                     |
|----------|--------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| `lat`    | number | TRUE     | Decimal representation of degrees latitude at the facility drawing's origin. The origin coordinates must be in WGS84 Web Mercator (EPSG:3857).  |
| `lon`    | number | TRUE     | Decimal representation of degrees longitude at the facility drawing's origin. The origin coordinates must be in WGS84 Web Mercator (EPSG:3857). |
| `angle`  | number | TRUE     | The clockwise angle, in degrees, between true north and the drawing's vertical (Y) axis.                                                        |

### Sample drawing package manifest

The JSON in this example shows the manifest file for the sample drawing package. Go to the [Sample drawing package] for Azure Maps Creator on GitHub to download the entire package.

#### Manifest file

```json
{
  "version": "2.0",
  "buildingLevels": {
    "dwgLayers": [
      "GROS$"
    ],
    "levels": [
      {
        "filename": "Ground.dwg",
        "levelName": "level 1",
        "ordinal": 0
      },
      {
        "filename": "Level_2.dwg",
        "levelName": "level 2",
        "ordinal": 1
      }
    ]
  },
  "georeference": {
    "lat": 47.63529901,
    "lon": -122.13355885,
    "angle": 0
  },
  "featureClasses": [
    {
      "featureClassName": "room",
      "dwgLayers": [
        "RM$"
      ],
      "featureClassProperties": [
        {
          "featureClassPropertyName": "name",
          "dwgLayers": [
            "A-IDEN-NUMR-EXST"
          ]
        },
        {
          "featureClassPropertyName": "roomType",
          "dwgLayers": [
            "A-IDEN-NAME-EXST"
          ]
        }
      ]
    },
    {
      "featureClassName": "wall",
      "dwgLayers": [
        "A-WALL-EXST",
        "A-WALL-CORE-EXST",
        "A-GLAZ-SILL-EXST",
        "A-GLAZ-SHEL-SILL-EXST",
        "A-GLAZ-SHEL-EXST",
        "A-GLAZ-EXST"
      ]
    },
    {
      "featureClassName": "workspace",
      "dwgLayers": [
        "A-BOMA"
      ]
    },
    {
      "featureClassName": "workspaceFurniture",
      "dwgLayers": [
        "A-FURN-SYTM-EXST"
      ]
    },
    {
      "featureClassName": "buildingFurniture",
      "dwgLayers": [
        "A-FURN-FREE-EXST"
      ]
    }
  ],
  "facilityName": "Contoso Building"
}
```

## Next steps

> [!div class="nextstepaction"]
> [Tutorial: Creating a Creator indoor map](tutorial-creator-indoor-maps.md)

Learn more by reading:

> [!div class="nextstepaction"]
> [Creator for indoor maps](creator-indoor-maps.md)

<!--    TBD: – NEED TO UPDATE LINKS TO THE NEW RELEASE OF THE CONVERSION SERVICE:  2023-03-01-preview    -->

[Conversion service]: https://aka.ms/creator-conversion
[Drawing package]: https://github.com/Azure-Samples/am-creator-indoor-data-examples
[Conversion Drawing Package Guide]: drawing-package-guide.md
[sample drawing package]: https://github.com/Azure-Samples/am-creator-indoor-data-examples
[onboarding tool example]: drawing-package-guide-2023-03-01-preview.md#
[feature class name]: drawing-package-guide-2023-03-01-preview.md#dwg-layers
[Georeference]: drawing-package-guide-2023-03-01-preview.md#georeference
[BuildingLevels]: #buildinglevels
[featureClass]: #featureclass
[featureClassProperty]: #featureclassproperty
