---
title: Unreal Engine Profiling
author: ipa
date: 2024-09-23
categories: [Unreal Engine]
tags: [ue, ue-editor, tools]
img_path: /unrealengine/profiling/
---

## Fundamentals

### Stat System

- `t.TargetFrameTimeThreshold <milliseconds>`: adjusts the Y-axis, also adjusts colors on `stat unit`

- `t.UnacceptableFrameTimeThreshold`: adjusts **red** color threshold on `stat unit`

- **stat commands**: triggers registered `EngineStats` function through `UEngine::HandleStatCommand`

  | Command             | Description                                                  | Configuration |
  | ------------------- | ------------------------------------------------------------ | ------------- |
  | `stat version`      | display build version string which includes the Changelist number | dev, test     |
  | `stat namedevent`   |                                                              | all           |
  | `stat fps`          | display average FPS                                          | all           |
  | `stat summary`      | display UsedPhysical memory stat                             | all           |
  | `stat unit`         | display frame times for Game Thread, Render Thread and GPU.  Also displays draw calls and primitive counts | all           |
  | `stat drawcount`    | display draw call counts per render pass                     | all           |
  | `stat hitches`      | display a scrolling hitch time when frame hitches occur      | all           |
  | `stat ai`           | display number of APlayerControllers and the number that were rendered to screen | all           |
  | `stat framecounter` | display the global, steadily increasing frame counter        | all           |
  | `stat levels`       | display a list of names of all the loaded levels             | all           |
  | `stat details`      | display detailed frame and fps timings                       | all           |
  | `stat unitmax`      | same as `stat unit`, but with max values also displayed      | dev, test     |
  | `stat unitgraph`    | displays a frame time stats graphed over time                | dev, test     |
  | `stat raw`          |                                                              | dev, test     |
  | `stat particleperf` |                                                              | dev, test     |
  | `stat tsr`          | same as `stat unit`, but with additional TSR settings        | dev, test     |
  | `stat streaming`*   | displays basic statistics on streaming assets, like how much memory streaming textures are using, or how many streaming textures there are in the scene | dev, test     |

  - the stat renders can be found in `RenderStat<Command>` functions, defined in <u>Engine.h</u>
  - all the `STATGROUP` names can be used as parameters of `stat` cmd to display the sub-stat values
    - the `STATGROUP` can be declared by `DECLARE_STATS_GROUP(...)`, ref: <u>Stats2.h</u>
  - ref: [Stat Commands in Unreal Engine](https://dev.epicgames.com/documentation/en-us/unreal-engine/stat-commands-in-unreal-engine?application_version=5.3)

- some other stat group commands:

  | Command                             | Description                                                  | Configuration |
  | ----------------------------------- | ------------------------------------------------------------ | ------------- |
  | `stat slow [-ms=1.0] [-depth=4]`    | show all the stats took longer than 1ms, up to hierarchy depth 4 | `STAT` macro  |
  | `stat dumphitches [-start] [-stop]` | flush the hitches to log during start and stop               | `STAT` macro  |
  | `profilegpuhitches`                 | handled by `UEngine::HandleProfileGPUHitchesCommand`         | dev, test     |

  - other cmd ref:
    - `StatCmd`: in StatsCommand.cpp
    - `UEngine::Exec`: in UnrealEngine.cpp
    - `UEngine::Exec_Dev`: in UnrealEngine.cpp

### FPS Commands

| Command          | Description                                                  |
| ---------------- | ------------------------------------------------------------ |
| `t.MaxFPS`       | Caps FPS to the given value.  Set to <= 0 to be uncapped (need to disable sync) |
| `r.SetFramePace` | Set a target frame rate for the frame pacer (sync with hardware's refresh rate, set to 0 to not sync at all, to fix tearing) |
| `r.VSync`        | Enable/Disable v-sync, should use together with frame pacer, but it's not the case till UE5.4, `-novsync` also works |

- `t.MaxFPS 0` won't be uncapping the FPS, unless **smooth frame rate is disabled**, ref: `UEngine::GetMaxTickRate`, either 

  - Disable smooth frame rate (**Project > Engine > General Settings**) disabled by default

  - Disable it in **DefaultEngine.ini**, (or override it at runtime, ref: [Override Configuration from the Command-line](https://dev.epicgames.com/documentation/en-us/unreal-engine/configuration-files-in-unreal-engine#overrideconfigurationfromthecommand-line)):

    ```ini
    [/Script/Engine.Engine]
    ; can be overriden by: `-ini:Engine:[/Script/Engine.Engine]:bForceDisableFrameRateSmoothing=1`
    bForceDisableFrameRateSmoothing=1
    ```

### Profiling Noises

List of launch parameters to reduce profiling noises:

| Parameter                       | Description                                                  |
| ------------------------------- | ------------------------------------------------------------ |
| `-novsync`                      | disable vsync at launch, same as `r.VSync 0`                 |
| `-noverifygc`                   | disable gc assumption about "Disregard For GC" objects and clusters, same as `gc.VerifyAssumptions 0`, on dev builds |
| `-nosound`                      | disable Wwise SoundEngine, will be no sound                  |
| `-noailogging`                  | disable AI logging                                           |
| `-benchmark`                    | set app to benchmark mode                                    |
| `-deterministic`                | set `-usefixedtimestep` and `-fixedseed` (ref: `FEngineLoop::PreInitPreStartupScreen`, random number generator init section) |
| `-fps=60`                       | set fps tp 60, which means the time step will be 16.67ms     |
| `-benchmarkseconds=180`         | set the duration for benchmark                               |
| `-corelimit=8`                  | limit the available cores for the game                       |
| `-ExecCmds="<cmd1>,<cmd2>,..."` | execute cmds at launch                                       |
| `D3D12.StablePowerState 1`      | enable stable power state. This increases GPU timing measurement accuracy but may decrease overall GPU clock rate, only works in developer mode, ref: [ID3D12Device::SetStablePowerState](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12device-setstablepowerstate#remarks) |

### Misc

Here are some helpful cmds to try during the profiling and analysis:

| Command                      | Description                                                  |
| ---------------------------- | ------------------------------------------------------------ |
| `pause`                      | Command to try to pause the game (this is a toggle)          |
| `r.screenpercentage <value>` | To render in lower resolution and upscale for better performance (combined up with the blendable post process setting). **70** is a good value for low aliasing and performance, can be verified with `show TestImage`, negative screen percentage is determined by `r.ScreenPercentage.Default` |

- cmds can be registered through `UFUNCTION(exec)` as `pause` defined as `APlayerController::Pause()`
  - ref: `UCheatManager` (in <u>CheatManager.h/.cpp</u>), can be accessed through the player controller

## Unreal Insights

### Basics (Launch Options)

General game launch options for profiling purposes:

```cmd
<GameProjectName>.exe [<mapname>][?game=<GameModeName>] -trace=default,task -nosound -noverifygc -novsync -execcmds="stat namedevents"
```

- for the game mode part, possibly need the `GameModeClassAlias`, ref: [GameMode via Commandline](https://forums.unrealengine.com/t/gamemode-via-commandline/317652/2)
- insight trace channels:
  - `default`: includes `cpu`,`gpu`,`frame`,`log`,`bookmark`,`screenshot`,`region`
  - `task`: to check the dependency between different task graphs/function calls
  - `loadtime`, `assetloadtime`
  - `memory`: includes `memtag`, `memalloc`, `callstack`, `module`
  - `metadata`, `assetmetadata`
  - `RDG`: for render dependencies
  - etc... (ref: [trace channels](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-insights-reference-in-unreal-engine-5#tracechannels))
- If none channel specified, the channels defined in `ini:Engine:[Trace.ChannelPresets]:Rendering` will be used:
  - by default: `gpu`, `cpu`, `frame`, `log`, `bookmark`


### Trace Commands (In Game Console)

| Command                            | Description                                                  |
| ---------------------------------- | ------------------------------------------------------------ |
| `Trace.Status`                     | Prints Trace status to console                               |
| `Trace.Start` / `Trace.File`       | Starts tracing to a file (the former is deprecated)          |
| `Trace.Stop`                       | Stops tracing profiling events                               |
| `Trace.Pause`                      | Pauses all trace channels currently sending events           |
| `Trace.Resume`                     | Resumes tracing that was previously paused (re-enables the paused channels) |
| `Trace.Enable [ChannelSet]`        | Enables a set of channels                                    |
| `Trace.Disable [ChannelSet]`       | Disables a set of channels                                   |
| `Trace.Bookmark [Name]`            | Emits a `TRACE_BOOKMARK(Name)` macro event with the given string name, `TraceBookmark` node in BP |
| `Trace.Screenshot [Name] [ShowUI]` | Takes a screenshot and saves it in the trace, in c++ use macro `TRACE_SCREENSHOT(...)` or use `FTraceScreenshot::RequestScreenshot(...)` |

### Customized Trace/Counter

*Can only added through c++ code*

| Macro                                                     | Description                                                  |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| `SCOPED_NAMED_EVENT(Name, Color)`                         |                                                              |
| `SCOPED_NAMED_EVENT_FSTRING(Text, Color)`                 | `Text` is a FString type, `Color` will be FColor type        |
| `DECLARE_SCOPE_CYCLE_COUNTER(CounterName, Stat, GroupId)` | no need to define `Stat` outside, but still need to define `GroupId` |
| `QUICK_SCOPE_CYCLE_COUNTER(Stat)`                         | the `CounterName` will be set to `#Stat`, `GroupId` will be `STATGROUP_Quick` |

All the macros can be found in <u>PlatformMisc.h</u> and <u>Stats2.h</u>

### Remote Profiling

#### Insight Connection

If the game is not running on the same device where the insight is installed, we can setup some connection configuration in the Insight session browser:

![connection](insight_connection_tab.png)

where:

- ***Trace recorder IP address***: this is the IP of the device where the Insight is found
- ***Running instance IP address***: this is the IP of the device where the game is actually running on
- ***Initial channels***: normally `default` is enough, check the previous section for more info on the channels

#### Session Frontend

This is another practical feature provided by Unreal through its another tool called: **Unreal Frontend**:

![session_frontend](session_frontend_tab.png)

- all local/remotely connected sessions will be displayed on the left side of the interface
- By selecting one, we can send command and even promote some commands as shortcut using the Console sub-tab. 

## External Tools

- **PresentMon**: [https://github.com/GameTechDev/PresentMon](https://github.com/GameTechDev/PresentMon)
- AMD: GPU profiler (GPU) / Ryzen Master (CPU)
- Nvidia: Nsight Graphics (GPU)
- Intel: XTU (CPU)
- MSI Afterburner (GPU)
- PIX for windows (GPU)