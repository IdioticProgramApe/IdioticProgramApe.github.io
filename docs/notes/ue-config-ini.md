---
layout: default
title: UE Config Ini
parent: Notes
nav_order: 2
---
<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Ini file system

If multiple ini files exists, and they happen to share a same key, then the new value will overwrite the old one.

In UE, there are many ini files and where store a lot of tuning parameter definitions, usually related to scalability.

The file hierarachy of these files are (sorted by load order), taking **Engine** config as example:

```text
* Engine/Config/Base.ini (usually empty)
* Engine/Config/BaseEngine.ini
* Engine/Config/<Platform>/<Platform>Engine.ini
* <ProjectDir>/Config/BaseEngine.ini
* <ProjectDir>/Config/<Platform>/<Platform>Engine.ini
* <ProjectDir>/Saved/Config/<Platform>/Engine.ini
```
