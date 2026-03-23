[TOC]

## Generating Trace Files

Enable named events:

- `-statnamedevents`: seems bugged, won't provide all the possible info unless an extra stat cmd executed in game
- `-execcmds="stat namedevents"`: OK! this is also the universal way of executing cmds at game launch



Start trace:

By default, if game launched with Unreal Insights opened, trace will be automatically started, otherwise use `trace.file [path] [channels]` in game to start:

- path: where this trace file will be stored at
- channels: [Unreal Insights Reference in Unreal Engine 5 \| Unreal Engine 5.7 Documentation | Epic Developer Community](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-insights-reference-in-unreal-engine-5#tracechannels)
- e.g. `trace.file default,memory`, path omitted



Stop trace:

- `trace.stop`: stop the trace file/profiling
- or simple shutdown the application



Other trace cmds:

- `trace.bookmark <name>`: to bookmark a moment during profiling
- `trace.screenshot <name>`: perform a screenshot and embed it into the trace file to provide more ingame info, however, the screenshot will add more overhead.



