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

### References

More details can be found on Unreal's official documentation:

- [Unreal Engine Build Tool Target Reference](https://docs.unrealengine.com/5.2/en-US/unreal-engine-build-tool-target-reference/)
- [Unreal Engine Modules](https://docs.unrealengine.com/5.2/en-US/unreal-engine-modules/)
- [Module Properties in Unreal Engine](https://docs.unrealengine.com/5.2/en-US/module-properties-in-unreal-engine/)
