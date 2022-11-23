---
layout: default
title: Reflection
parent: Meta-Programming
grand_parent: Python
nav_order: 7
permalink: /docs/python/meta-programming/reflection/
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Definition

The reflection is pure metaprogramming (not only used for object oriented based metaprogramming), it's an ability of a program to view and modify its own structure at runtime.

> Note: Not always true, but in python, if you think something might be possible, then it probably is and there is probably a function for it.

## Application - A Runtime Debugger

### Step Over

We want our code to run every time a new line is executed by python

- use `sys.settrace(func)`
  - `func` should take in `(frame, event, arg)`
  - `func` will be called everytime a function a called
  - `func` can return another function
    - this returned function also takes in `(frame, event, arg)`
    - will be called everytime python executes the next line
    - need not return anyt