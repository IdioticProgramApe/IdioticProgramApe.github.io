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
- `LevelStreaming`, `WorldComposition` before UE5.0, after use `WorldPartition` 
- prevent gc:
  - `FReferenceCollector::AddReferencedObject`
- texture streaming
  - `FStreamingRenderAsset::UpdateRetentionPriority_Async`
  - 