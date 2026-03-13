---
title: Unreal Engine GPU Optimization
author: ipa
date: 2026-03-12
categories: [Unreal Engine]
tags: [ue, wip]
---

## Check List

| Subject       | Debug Methods                                                | Potential Solutions                                          |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Overview      | - cmd: `stat gpu`                                            |                                                              |
| Lights        | - light complexity viewmode                                  | - uncheck `Affects World` property of lights                 |
| VSM           | - vsm cached pages viewmode<br />- cmd: `r.shadow.virtual.showstats 1`, check cached stats<br />  - make sure `r.shaderprint` is enabled | - further check WPO<br />                                    |
| WPO           | - nanite WPO viewmode                                        | - uncheck `Evaluate World Position offset` for static meshes<br />- contact shadows can be causing WPO being evaluated (in the Materials)<br />- properly setup the evaluation distance (only evaluate at close) |
| Particles     | - `Shift + /`, reset all instances in the world<br />- cmd: `stat niagarasystemcounts`: to check ns instance info<br />- cmd: `stat niagarasystems`: to check ns cost<br />- cmd: `stat niagara`: to get more info | - culling problems<br />  - no `Effect Type` configured in the ns (ref: `UNiagaraEffectType`)<br />    - setup a considerate `Update Frequency`<br />    - `System Scalability Settings`: `Visibility Culling`<br />  - individual culling in each render <br />  - GPU or CPU particles ? (check the bottleneck) |
| Assets        | - Tools -> Audit -> Statistics                               | - check the assets which only have one instance in the world<br />  - do we want them ? can be deleted ?<br />- check the sizes<br />  - enable nanite on hi-poly can reduce sizes |
| Static Meshes | - Static Mesh Editor                                         | - check the triangle count, should nanite enabled or disabled ?<br />  - if is nanite, check fall back triangles -> subject nanite |
| Nanite        | - cmd: `nanitestats`: check shading bins stats (draw calls)<br />  - cmd: `r.nanite.showstats 0`: to disable the stats view<br />- cmd: `showflag.nanitemeshes` to view all non-nanite meshes<br />  - alternative: nanite mask viewmode <br />- nanite triangle visualization viewmode | - can be impacted by the Asset audit, if unique instances get removed<br />- keep the usage of `Material Instance Dynamic` to a moderate level<br />  - alternative: `Custom Primitive Data` (doesn't require unique materials ?) |
| Collision     | - visibility collision viewmode<br />  - might need to hide meshes to have a better view | - do we need a collision with many details ?                 |

TODO: 37:24

