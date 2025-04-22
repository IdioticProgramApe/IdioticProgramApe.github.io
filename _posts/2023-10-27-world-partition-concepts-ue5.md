---
title: World Partition System Concepts
author: ipa
date: 2023-10-27
categories: [Unreal Engine]
tags: [ue, ue-editor]
img_path: /unrealengine/worldpartition/
---

## World Partition

- World partition is the new world streaming system in UE5: 

  - [World Partition in Unreal Engine \| Unreal Engine 5.5 Documentation \| Epic Developer Community](https://dev.epicgames.com/documentation/en-us/unreal-engine/world-partition-in-unreal-engine)
  - [World Partition Series](https://www.youtube.com/playlist?list=PLtXHb7xLAWqsIBp8HNIggOAVjzDiUeqV7)

- Regions of the world called `cells` are streamed in and out of view visibility based on the distance from their streaming source, usually the player.

  ![cell_manipulations](world_partition_01.png)
  _world partition cell manipulations_

- World partition replaces UE4's world composition feature and its level streaming system, there are 3 ways to enable world partition in UE5:

  - creating a new project from a template in the `Games` category

  - creating a new level using the `Open World` template

  - converting an existing level using:

    - `Tools` -> `Convert Level...` -> 

    - world partition convert commandlet:

      ```bat
      UnrealEditor.exe ProjetName -run=WorldPartitionConvertCommandlet MapName.umap -AllowCommandletRendering
      ```
  
- Some important console variables, defined in `WorldPartitionSubsystem.cpp`:

  | Console Variable                         | Description                                                  | Default Value |
  | ---------------------------------------- | ------------------------------------------------------------ | ------------- |
  | wp.Runtime.ToggleDrawRuntimeHash2D       | Toggles 2D debug display of world partition runtime hash     | 0             |
  | wp.Runtime.ToggleDrawStreamingSources    | Toggles debug display of world partition streaming sources   | 0             |
  | wp.Runtime.ToggleDrawStreamingPerfs      | Toggles debug display of world partition streaming perfs     | 0             |
  | wp.Runtime.ToggleDrawRuntimeCellsDetails | Toggles debug display of world partition runtime streaming cells | 0             |
  | wp.Runtime.ToggleDrawDataLayers          | Toggles debug display of active data layers                  | 0             |
  | wp.Runtime.ToggleDrawDataLayersLoadTime  | Toggles debug display of active data layers load time        | 0             |
  | wp.DumpstreamingSources                  | Dumps active streaming sources to the log                    |               |
  | wp.Runtime.UpdateStreamingStateTimeLimit | Maximum amount of time to spend doing World Partition UpdateStreamingState (ms per frame) | 0.f           |

  

## Landscape

### Landscape Properties

- World composition disables the **landscape location, rotation and scale properties**, 
- With world partition, these properties can still be adjusted even after the terrain has been imported. This allows for changing the terrain elevation or scaling at any time during level development

### Landscape Sizes

- Recommended landscape sizes: [Landscape Technical Guide in Unreal Engine \| Unreal Engine 5.5 Documentation \| Epic Developer Community](https://dev.epicgames.com/documentation/en-us/unreal-engine/landscape-technical-guide-in-unreal-engine#recommendedlandscapesizes)

- some other intermediate sizes are also working well with unreal: 
  - 1513 x 1513
  - 3025 x 3025
  - 6097 x 6097
- All the sizes can be directly chosen with **landscape create mode**
- To create landscape larger than 8129 x 8129, should use the recommended world partition large sizes for PNG files. The recommended world partition large sizes are different from the world composition large sizes.

### Landscape Importer

- With world partition, no longer have to split the heightmaps into tiles for importing, possible to import the full PNG heightmap file for resolutions up to 32641x32641
  - for those larger than 32641, tiles have to be used.

### Landscape Tile Sizes

- The unreal engine landscape importer **ignores the shared edges**, and loads each tile as straight fit edge to edge:
  - a 10x10 tile set with 1009x1009 size tiles original resolution: **10081x10081** (= 10x1009 - 9)
  - after imported into unreal engine, resolution became: **10090x10090** (= 10x1009)
  - since 10090 is not divisible by the new larger **511** component size, world partition pads out the terrain size to **10201x10201**
- if not using proper world partition sizing for tiles, possible will end up with the cropped heightmap or padded heightmap
- one method to determine the tile sizings for various world partition heightmaps is to **check the resolution is a prime number**, to get the proper tile sizing:
  - 16321 = 19 x 859
  - 24481 = ?
  - 28564 = 13 x 2197 = 169 x 169
  - 32641 = 7 x 4663
  - 44881 = 37 x 1213
  - 48961 = 11 x 4451

## One File Per Actor (OFPA)

- The OFPA system stores the actor assets and properties into the `__ExternalActors__` folder in the project content folder.
- Landscape actors appear always to be set to the external actor mode, and stored in the `__ExternalActors__` folder
- The OFPA system will be used even for small single heightmap landscapes such as 4033 and 8129.
  - 8129x8129 landscape heightmap import will result in **1 landscape actor** and **256 landscape streaming proxy actors** in the outliner list.
- The landscape streaming proxy actors and resultant heightmap uasset files will be stored in the `__ExternalActors__` folder, this results some issues which need to be considered, see in the Issue section.

## System Requirements

| World Size  | System Memory | Video Memory  |
| ----------- | ------------- | ------------- |
| 8km x 8km   | 32GB          | 8GB RTX-3050  |
| 16km x 16km | 64GB          | 10GB RTX-3080 |
| 32km x 32km | 128GB         | 24GB RTX-3090 |
| 64km x 64km | 512GB         | 48GB RTX-8000 |

* values are benchmarked, can vary on different machines

## Cells

### Cell Size

- The cell size determines the size of the grid cells that are used to generate the streaming levels.
- The default cell size is 25600cm x 25600cm = 256m x 256m
  - this property can be adjusted in: World Settings -> World Partition -> Runtime Settings -> Cell Size
  - The cell grid can be viewed in the viewport **by enabling the Preview Grids** in the world partition runtime settings
  - Each cell would be close to the same size as the landscape component size of 8129x8129 landscape, which is 256x256
- A 8129x8129 landscape is a 1024 x 4 = 4096 draw calls for the entire terrain, a cell loading range of 76800 with 25600 cell size results in a landscape draw call count of about 50 draw calls.
  - for larger landscapes, the component size is increased to 511, mostly reduce the number of draw calls per landscape.

### Cell Loading Range

- The cell loading range is the distance from the streaming source which is typically the player that cells in the world will be loaded and made visible.

- For levels that also contain the landscapes, this loading range would include the landscape streaming proxy actors that are visible and rendered in the view distance.

- The cell loading range can be set: World Settings -> World Partition -> Runtime Settings -> Loading Range

  - by default it's set to 76800cm = 768m

  - the default loading range is usually too small for most landscape world designs where **a visual distance of ~5km is optimal**, which will require the cell loading range to be 512000cm = **5120m**

  - some benchmarks with a loading range of 5km:

    | CPU            | GPU      | Resolution | Mode       | FPS  |
    | -------------- | -------- | ---------- | ---------- | ---- |
    | Ryzen9 5950x   | RTX-3090 | 1920x1080  | PIE        | 120  |
    | Intel i3 12100 | GTX-1650 | 1920x1080  | Standalone | 30   |
    | Intel i3 12100 | GTX-1650 | 1280x720   | Standalone | 40   |
    | Intel i3 12100 | GTX-1650 | 640x360    | Standalone | 60   |

## MiniMap

### World Partition MiniMap

- The world partition minimap allows to select and load the individual cells of the landscape
- The minimap is divided into landscape sections:
  - for 8129x8129 landscape which has 2x2 sections for 32x32 components, there will be a grid of 64x64 sections that can be selected, loaded or unloaded
  - the viewport render **performance may drop significantly** while the world partition minimap tab is selected

### MiniMap Texture

- The minimap is the top-down overview of the world that is used to control loading and unloading of cells and the display of placed actors in the world

- A 16k x 16k minimap is the largest minimap that can be created with default texture sizes:

  - creating a 16k x 16k minimap texture for a 16km x 16km world usually causes the texture streaming pool to be over budget, requiring that pool to be increased

  - attempting to create a minimap texture larger than 16k with standard textures results in the following log error

    ![too_large_minimap_texture](world_partition_02.png)
    _oversized minimap log errors_

  - if **virtual texture are enabled** in the project properties then larger minimaps can be created such as for a 32km x 32km world

- The minimap texture counts towards the computer system texture streaming pool

  - system can run out of pool resources if large minimap textures are created

## Hierarchical LOD (HLOD)

- source: [World Partition - Hierarchical Level of Detail in Unreal Engine \| Unreal Engine 5.5 Documentation \| Epic Developer Community](https://dev.epicgames.com/documentation/en-us/unreal-engine/world-partition---hierarchical-level-of-detail-in-unreal-engine)
- Some world components, such as **Landscapes** and **Water components**, are currently **not supported** by HLOD Actors
  - no use for a test map only contains landscape

## Actor Visibility

- use the `Is Spatially Loaded` checkbox option in the actors details world partition settings to control the persistent visibility of the actor, if it's unchecked, the actor will always be visible even if the world partitioned cell that it sits on is unloaded.

## Issues

- It appears that sometimes the landscape streaming proxies in landscape actor cannot be deleted from an existing level, this deletion takes a long time even on a fast computer and is usually unsuccessful:
  - the landscape streaming proxy actors will often reappear in the outliner list after reopening the level
  - the project size also shows that the external actors OFPA folders did not decrease in size -> GB of orphaned files with no OFPA clean function in UE5
  - need to delete the entire level
