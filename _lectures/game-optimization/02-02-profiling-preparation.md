## Prepare Game Build

PIE behaves differently than the actual game builds in terms of the performance.

However, sometimes there are still some modes could be useful to get some information.



**Standalone Game**

Make sure to enable the real-time **DISABLED** in the viewport, and minimize the editor. 

This is to make sure the running editor has minimum impact on the profiling.

Still this is not for something accurate.



**Packaged Build**

No need to package all maps, consider use Unreal Frontend or UAT command lines

- game configuration: 
  - `DebugGame`: **NO** 
  - `Development`: provides most in-depth profiling information (but all optimization applied)
  - `Shipping`: ideal for the final optimizations, no profiling is allowed
  - `Test`: only available from source code of the engine (shipping optimization + console & tracing)
    - no stat named events (less markers)
- project settings:
  - `smooth frame rate`: should be disabled
  - `v sync`: should be disabled, `-novsync` / cvar `r.vsync 0`
  - `max fps`: should be disabled, cvar `t.maxfps 0`



## Reducing Profiling Noise



A example of build command

```bat
start MyGame.exe ^
/Game/.../MyTestLevel?game=MyGameMode ^
-trace=default ^
-benchmark ^
-deterministic ^
-fps=60 ^
-benchmarksecond=180 ^
-novsync ^
-noverifygc
```



| Parameter             | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| `-novsync`            | avoid v-sync to impact the game's frame pacing to get an accurate fps number |
| `-noverifygc`         | eliminate gc overhead to get accurate gc numbers             |
| `-nosound`            | disable sound, if sound is not to be profiled                |
| `-noailogging`        | disable ai logging overhead, keep performance accurate       |
| `-benchmark`          | only exists in dev builds                                    |
| `-deterministic`      | equivalent to `-usedfixedtimestep` and `-fixedseed`          |
| `-fps=xx`             | not limit the frame rate, provides a fake delta time as (1000/xx) ms |
| `-benchmarksecond=xx` | define how many frames it would profile before closing<br />in this example: 60 * 180 = 10800 frames |

**Fixed seed**

``` c++
// Initialize random number generator.
{
    uint32 Seed1 = 0;
    uint32 Seed2 = 0;

    if (!FApp::bUseFixedSeed)
    {
        Seed1 = FPlatformTime::Cycles();
        Seed2 = FPlatformTime::Cycles();
    }

    FMath::RandInit(Seed1);
    FMath::SRandInit(Seed2);

    UE_LOG(LogInit, Verbose, TEXT("RandInit(%d) SRandInit(%d)."), Seed1, Seed2);
}
```



**Stat Cmds**

"live viewing" of stat commands add overheads to the performance results 



**Spline or static camera locations**

Make a special game mode and launch the game with this game mode



**In-game gameplay playback**

`demorec` allows to record a in-game gameplay, requires the game multiplayer supported, since it uses replication



**CPU cores**

Use `-corelimit=6` to get closer to target hardware core counts when profiling. 

This may help identify multithreading issues earlier as workstations might have significantly more core than target hardware.

Ref: [Steam HW Survey (CPU) results](https://store.steampowered.com/hwsurvey/cpus/), most users have between 4 and 8 cores.



**GPU clock**

To increase run-to-run timing accuracy, can **lock the frequencies of the GPU**. They will generally use lower than their dynamic clock speeds, but this will increase consistency. 

For DX12 this is directly available without third party tools in Unreal! `D3D12.StablePowerState 1`

*"If true, enable stable power state. This increases GPU timing measurement accuracy but may decrease overall GPU clock rate."*



## Launch Game

Create several shortcut of the game executable located in **MyGame/Binaries/Win64**.

Open the properties of the shortcuts, modify the `Target` field, with the launch parameters required for this particular game launcher. 

