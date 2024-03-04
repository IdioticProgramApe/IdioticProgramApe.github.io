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
- set a ref param to a function used in blueprint:

  ```c++
  UFUNCTION(BlueprintCallable)
  void SomeFun(PARAM(Ref)int& SomeInteger, PARAM(Ref)AActor* Actor)
  ```
- a `delay` node with delay set to `0.0` will execute its output next frame
- `QUICK_SCOPE_CYCLE_COUNTER` to quickly add a scope for the profiling in both Frontend and Insights
- UE4 async action methods, [https://xusjtuer.github.io/post/ue4-post8_async_bp_node_set_timer/](https://xusjtuer.github.io/post/ue4-post8_async_bp_node_set_timer/):

  - In blueprint
    - `Delay`
    - `Timeline`
    - `SetTimerByFunctionName`/`SetTimerByEvent`
    - `Tick`
  - In C++
    - `UFUNCTION`: `Latent` keyword, need an instance from `FPendingLatentAction`, i.e. `Delay`, `MoveComponentTo`
    - `UBlueprintAsyncActionBase`, i.e. `DownloadImage`, [Creating Asynchronous Blueprint Nodes](https://nerivec.github.io/old-ue4-wiki/pages/creating-asynchronous-blueprint-nodes.html)
    - `Tick`
- **Unreal Insight** common flags: 

  - `-tracehost=... -trace=cpu,log,bookmark,frame,memory,loadtime,rhicommands,rendercommands`
  
- **Unreal TextureGroupSettings** are initialized in `FRenderAssetStreamingManager`'s ctor:

  - to get the current device profile settings:

    ```c++
    const FTextureLODGroup& TexGroup = UDeviceProfileManager::Get().GetActiveProfile()->GetTextureLODSettings()->GetTextureLODGroup(TextureGroup(LODGroup));
    ```

- Quickly find the first instance of type **T** in the current world:

  - to get the current world: 

    ```c++
    UWorld* World = GEngine->GetWorldFromContextObject(WorldContextObject, EGetWorldErrorMode::LogAndReturnNull);
    ```

  - to find first object of type **T** from the world (standard way):
  
    ```c++
    for (TActorIterator<T> It(World); It; ++It) { return *It; }
    ```
  
  -  there is a transient member variable which may come into handy: `UWorld::PerModuleDataObjects`, can start searching from there.
  
- Useful commands in UE to get a gameplay profiling data:

  - before: 
    - `t.FSPChart.DoCsvProfile 1`: enable csv profile recording along with fps chart
    - `stat fps`: add fps info into the profiling data
    - `stat unitmax`: add other frametime and memory info from a global perspective into the profiling data
  - start/stop:
    - `StartFPSChart`: before the gameplay
    - `StopFPSChart`: after the gameplay
  - collect the data:
    - the output folder will be: *<project_dir>/Saved/Profiling/FPSChartStats*
