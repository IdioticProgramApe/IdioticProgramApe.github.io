[TOC]

## Frame Time (ms) vs. Frame Rate (fps)

| Name       | Unit     | Use Case                                        |
| ---------- | -------- | ----------------------------------------------- |
| Fame Time  | ms       | describe time consumption, used in optimization |
| Frame Rate | fps / Hz | set targets i.e. 30 fps, 60 fps                 |

From the following image, we can see that:

![frametime-framerate](https://www.intel.com/content/dam/developer/articles/training/unreal-engine-4-optimization-tutorial/img1.png)

Frame rate and frame time are inversely proportional $\displaystyle\frac{1}{FPS} \cdot 1000 = MS$, by just giving out how many fps we save from an optimization, we cannot know how much time we save, save 30 fps it could be:

- 16.67 ms (30 fps -> 60 fps)
- 5.56 ms (60 fps -> 90 fps)
- 2.77 ms (90 fps -> 120 fps)



## Basics of Frame Pacing

Frame rate describe how many frame rendered in one second in game, which is an average number, frame pacing on the other hand describes how consistent the frames are.

Having a good frame rate doesn't mean have a smooth user feeling experience.



| Console                                 | Note                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| `stat unitgraph`                        | in game performance monitor                                  |
| `stat raw`                              | enable unfiltered stats on unit graph                        |
| `t.targetframetimethreshold <ms>`       | adjust the unit graph y-axis as well as the target frame time |
| `t.unacceptableframetimethreshold <ms>` | beyond which the performance value will be red               |



Some common causes affecting the frame pacing:

- running slow code on lower frequencies, using timers or lower frequency ticks
- waiting on expensive ops, not using async ops:
  - physics line traces: async line trace -> act on the results at the beginning of the next frame
- creating too many GPU resources in one frame could stall the game
  - streaming, add too many lights (especially cast shadows)
- JIT compiling PSOs (DX12) 
- running garbage collecting (GC)
  - avoid gc, i.e. object pooling (memory overhead)
- spawning/streaming too many object at once



## (CPU vs. GPU) Performance Bottleneck

There several stats from `stat unit` provided by UE to describe the overall game performance

| Name | Notes                                                   |
| ---- | ------------------------------------------------------- |
| Game | CPU Game Thread: Blueprint, C++, AI, Slate (UI), etc... |
| Draw | CPU Render Thread: process graphical scene (changes)    |
| RHIT | CPU RHI Thread: Process Draw Commands to GPU            |
| GPU  | GPU: Execute Draw Commands from RHI                     |

- Profile on different spot in a level to get full pic of the overall performance
- Profile on relevant hardware not the workstation (which is usually not the targeted devices)
- Profile with desired scalability settings (low/medium/high/epic)
- Profile with vsync disable `r.vsync 0` and unlimited fps `t.maxfps 0`



Settings help to diagnose where the bottleneck is:

- `r.screenpercentage <pct>`: to check GPU bottleneck
  - some upscaler techs will impact on this
- `pause`: pause the GT, to check game is CPU/GPU bound



Third-Party performance overlays are likely inaccurate, because UE is unlikely to use all the device resources 