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
- `trace.screenshot <name> <showui>`: perform a screenshot and embed it into the trace file to provide more in-game info, however, the screenshot will add more overhead.



C++ macros

- `TRACE_BOOKMARK(...)`: same as cmd `trace.bookmark`
- `TRACE_SCREENSHOT(...)`: same as cmd `trace.screenshot`



C++ functions

- `FTraceScreenshot::RequestScreenshot(FString Name, bool bShowUI);`



BP nodes

- `TraceBookmark`
- `ExecuteConsoleCommand` with `trace.bookmark` or `trace.screenshot`



## Tracking Individual Stats (Graph Track)

UE version: 5.6 Plot Timer

Right click on one item needs investigation in the **Timers** view-> **Plot Timer** -> 

- Add instance series to graph task (time cost per call): check whether the function call itself is too expensive
- Add game frame stats series to graph track (time cost per frame): check whether the function call too many (each call not very expensive, together with instance series)



Main Graph: turn on/off in All Tracks in the tracks view

- Use `Shift + Scroll Wheel` to change the time scale in the main graph to be able to show the info

- Right click on the graph to customize the graph



## Adding Customized Traces & Counters

C++ macros to trace time

- `SCOPED_NAMED_EVENT(Name, Color)`
- `SCOPED_NAMED_EVENT_FSTRING(Name, Color)` (need it at runtime)



C++ macros to trace counter

- `TRACE_DECLARE_INT_COUNTER(CounterName, CounterDisplayName)`
  - to increment the counter: `TRACE_COUNTER_INCREMENT(CounterName)`
  - to decrement the counter: `TRACE_COUNTER_DECREMENT(CounterName)`
- `DECLARE_SCOPE_CYCLE_COUNTER(DisplayName, StatName, StatGroup)`
  - to actually recording the counter in a scope: `SCOPE_CYCLE_COUNTER(StatName)`
  - to show the stat on-screen: `stat <StatGroup>`



Will add overheads.



## Profiling on Steam Deck

Tech Spec: [Steam Deck :: Tech Specs](https://www.steamdeck.com/en/tech)

| Name               | Details                                                      |
| ------------------ | ------------------------------------------------------------ |
| APU                | 6 nm AMD APU<br/>CPU: Zen 2 4c/8t, 2.4-3.5GHz (up to 448 GFlops FP32)<br/>GPU: 8 RDNA 2 CUs, 1.6GHz (1.6 TFlops FP32)<br/>APU power: 4-15W |
| RAM                | 16 GB LPDDR5 on-board RAM (6400 MT/s quad 32-bit channels)   |
| Storage            | Steam Deck 512GB NVMe SSD<br/>Steam Deck 1TB NVMe SSD<br/>Both include high-speed microSD card slot |
| Gamepad controls   | A B X Y buttons<br/>D-pad<br/>L & R analog triggers<br/>L & R bumpers<br/>View & Menu buttons<br/>4 x assignable grip buttons |
| Thumbsticks        | 2 x full-size analog sticks with capacitive touch            |
| Haptics            | HD haptics                                                   |
| Resolution         | 1280 x 800 x RGB                                             |
| Maximum Brightness | 1,000 nits peak brightness (HDR)<br/>600 nits (SDR)          |
| Refresh rate       | up to 90Hz                                                   |
| Desktop            | KDE Plasma                                                   |



Epic doc: [Steam Deck Quick Start in Unreal Engine](https://dev.epicgames.com/documentation/en-us/unreal-engine/steam-deck-quick-start-in-unreal-engine)

Tools:

- **SteamOS Devkit Client** (from steam or direct install): [How to load and run games on Steam Deck (Steamworks Documentation)](https://partner.steamgames.com/doc/steamdeck/loadgames)
- **Unreal Insights**
- **Unreal Session Frontend**



Proton version might need to be modified



Unreal Insights

- set Running Instance IP address to the steam deck IP
- specify the trace channels
- after game launched on the deck, click on Connect to hook the insights to the remote game



Frontend

- the deck game session will be listed in the Unreal Session Frontend
- console view will output the game logs and support sending commands to deck (also can promote cmd to shortcuts to avoid repeating input)
