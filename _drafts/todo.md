## Unreal Engine

- how to set up a UE project with Clion

  - [Unreal Engine + CLion Integration - YouTube](https://www.youtube.com/watch?v=VjeYOceslZE)
  - plugin: `CLionSourceCodeAccess`, to provide 2 more options in `File`: `Generate CMakeList` and `Open CLion`
  - project settings: `CMake Target Configuration` for CLion and more
  - in CLion workspace view, right click on the `cmake-build-debug` and click on `reload cmake project`, once a new cmake file is generated
  - and it crashes all the time.
- `UBlueprintLibrary` (to expose python functions, later on with `-run=pythonscript` to launch python files)
- `Slate` to create tools (have a nice UI widget)
- `AutoTest`
- `ECS`: [Entity Component System (guru99.com)](https://www.guru99.com/entity-component-system.html), [The Entity-Component-System (gamedeveloper.com)](https://www.gamedeveloper.com/design/the-entity-component-system---an-awesome-game-design-pattern-in-c-part-1-)
- `LevelStreaming`, `WorldComposition` before UE5.0, after use `WorldPartition` 
- prevent gc:

  - `FReferenceCollector::AddReferencedObject`
- texture streaming

  - `FStreamingRenderAsset::UpdateRetentionPriority_Async`
- light baking: [Important Tips for Baking Light in Unreal Engine - YouTube](https://www.youtube.com/watch?v=fbSEY-QjM4g)

  - PBR
  - Base colors
  - Actor & Light Mobility

    | Actor      | Light      | Direct Diffuse | Indirect Diffuse | Shadows   |
    | ---------- | ---------- | -------------- | ---------------- | --------- |
    | Static     | Static     | Baked          | Baked            | Baked     |
    | Static     | Stationary | Baked          | Baked            | Real-Time |
    | Static     | Moveable   | Real-Time      | None             | Real-Time |
    | Stationary | Static     | Volume Samples | Volume Samples   | None      |
    | Stationary | Stationary | Real-Time      | Volume Samples   | Real-Time |
    | Stationary | Moveable   | Real-Time      | None             | Real-Time |
    | Moveable   | Static     | Volume Samples | Volume Samples   |           |
    | Moveable   | Stationary | Real-Time      | Volume Samples   | Real-Time |
    | Moveable   | Moveable   | Real-Time      | Volume Samples   | Real-Time |

  - ...
- memory

  - skeletal mesh: for `UMA` (Uniform Memory Access) system, no need to copy data from cpu to gpu, they can both have access to some amount of data.
    - UE: `Allow CPUAccess` in `LOD` section in Skeletal Mesh window.
  - leak: [在虚幻引擎 4 中处理内存泄漏问题 (unrealengine.com)](https://www.unrealengine.com/zh-CN/tech-blog/dealing-with-memory-leaks-in-ue4)
    - modify `#define MALLOC_LEAKDETECTION 1` in MallocLeakDetection.h and recompile
    - `MallocLeak Start` to start recording, `MallocLeak Stop` to stop recording
      - use `MallocLeak Dump N` to filter out at least N bytes allocations
- check if in PIE mode:

  -  `GIsEditor && FApp::IsGame()`