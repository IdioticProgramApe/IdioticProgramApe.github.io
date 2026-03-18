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