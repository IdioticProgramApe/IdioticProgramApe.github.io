---
layout: default
title: Inline Function (C11)
parent: C
grand_parent: C/C++
nav_order: 5
permalink: /docs/c-c++/c/inline-function/
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Declarations

### Single File Program

- Declare an inline function by placing the keyword inline before the function declaration

  ```c
  inline void some_func();
  ```

- There are different places to create inline function definitions (same file or header file)

- For the compiler to make inline optimizations, it has to know the contents of the function definition

- The inline function definition has to be in the same file as the function call (**internal linkage**)

- Should always use the **inline** function specifier along with the **static** storage-class specifier (using extern less portable)

  - inline functions are usually defined before their first use in a file (definition also acts as a prototype)

  ```c
  inline static void foo() //inline definition/prototype
  {
      // do something ... 
  }
  ```

### Multi-file Program

- For a multi-file program, need an inline definition in each file that calls the function
  - the simplest way to accomplish this is to put the inline function definition in a header file
  - include the header file in those files that use the function
- An inline function is an exception to the rule of not placing executable code in a header file
  - because the inline function has **internal linkage**, defining one in several files does not cause problems

## Note

- The inline declaration is only advice to the compiler, which can decide to ignore it
  - may cause the compiler to replace the function call with inline code and/or perform some other sorts of optimizations, or it may have no effect

