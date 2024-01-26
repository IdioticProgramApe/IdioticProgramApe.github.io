---
title: Targets & Modules in Unreal
author: ipa
date: 2023-09-15
categories: [Unreal Engine]
tags: [ue, ue-editor]
---

## Unreal Engine UBT

For an unreal c++ project, the building process are covered by a customized tool. It can manage to make the build adaptive to all kinds of configurations.

### Unreal Engine Targets

In UE, each target will need a **[TargetName].Target.cs** to micromanage the module dependency and some other extra information we want the UBT to get when build the corresponding target.

Each target can have multiple modules.

### Unreal Engine Modules

TL;DR: **Modules** are the basic building block of **Unreal Engine's** software architecture. These *encapsulate* specific editor tools, runtime features, libraries, or other functionality in standalone units of code.



In order to extend the editor, we need to create our own modules. The takeaway points for creating the modules:

- modules enforce good code separation:
  - prevent the original engine code from being accidently polluted by our own changes
  - also provide good inter-module communications
- all modules require a **[moduleName].Build.cs**, for ex. the uproject itself is a module.
- outside modules can be included by adding them to the Build.cs file as dependencies

### Unreal Plugins

One unreal plugin can have multiple different modules, to work together to accomplish some unreal features.

For each one of the modules reside inside of a plugin, there are 2 properties to specify in the uplugin file:

```json
{
    //...
    "Modules": [
        {
            "Name": "UnrealEdExtension",
            "Type": "Editor",
            "LoadingPhase": "PreDefault"
        }
    ]
}
```

#### Module Type

Here is list of module types available to choose from for the module configuration:

| Type Name                | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| `Runtime`                | Loads on all targets, except programs.                       |
| `RuntimeNoCommandlet`    | Loads on all targets, except programs and the editor running commandlets. |
| `RuntimeAndProgram`      | Loads on all targets, including supported programs.          |
| `CookedOnly`             | Loads only in cooked games.                                  |
| `UncookedOnly`           | Only loads in uncooked games.                                |
| `Developer`              | Deprecated due to ambiguities. Only loads in editor and program targets, but loads in any editor mode (eg. -game, -server). Use `UncookedOnly` for the same behavior (eg. for editor blueprint nodes needed in uncooked games), or `DeveloperTool` for modules that can also be loaded in cooked games but should not be shipped (eg. debugging utilities). |
| `DeveloperTool`          | Loads on any targets where bBuildDeveloperTools is enabled.  |
| `Editor`                 | Loads only when the editor is starting up.                   |
| `EditorNoCommandlet`     | Loads only when the editor is starting up, but not in commandlet mode. |
| `EditorAndProgram`       | Loads only on editor and program targets                     |
| `Program`                | Only loads on program targets.                               |
| `ServerOnly`             | Loads on all targets except dedicated clients.               |
| `ClientOnly`             | Loads on all targets except dedicated servers.               |
| `ClientOnlyNoCommandlet` | Loads in editor and client but not in commandlets.           |

For the most of the time, `Runtime` is for Game and `Editor` is for the unreal editor.

#### Module Loading Phase

Here is list of module loading phase available to choose from for the module configuration:

| Loading Phase           | Description                                                  |
| ----------------------- | ------------------------------------------------------------ |
| `EarliestPossible`      | As soon as possible - in other words, uplugin files are loadable from a pak file (as well as right after PlatformFile is set up in case pak files aren't used) Used for plugins needed to read files (compression formats, etc) |
| `PostConfigInit`        | Loaded before the engine is fully initialized, immediately after the config system has been initialized.  Necessary only for very low-level hooks |
| `PostSplashScreen`      | The first screen to be rendered after system splash screen   |
| `PreEarlyLoadingScreen` | Loaded before core UObject for setting up manual loading screens, used for our chunk patching system |
| `PreLoadingScreen`      | Loaded before the engine is fully initialized for modules that need to hook into the loading screen before it triggers |
| `PreDefault`            | Right before the default phase                               |
| `Default`               | Loaded at the default loading point during startup (during engine init, after game modules are loaded.) |
| `PostDefault`           | Right after the default phase                                |
| `PostEngineInit`        | After the engine has been initialized                        |
| `None`                  | Do not automatically load this module                        |

Similarly, most of the time, `Default` for game and `PreDefault` for the editor.

### References

More details can be found on UE's official documentation:

- [Unreal Engine Build Tool Target Reference](https://docs.unrealengine.com/5.2/en-US/unreal-engine-build-tool-target-reference/)
- [Unreal Engine Modules](https://docs.unrealengine.com/5.2/en-US/unreal-engine-modules/)
- [Module Properties in Unreal Engine](https://docs.unrealengine.com/5.2/en-US/module-properties-in-unreal-engine/)
