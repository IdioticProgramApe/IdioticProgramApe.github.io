---
layout: default
title: Noreturn Function (C11)
parent: C
grand_parent: C/C++
nav_order: 6
permalink: /docs/c-c++/c/noreturn-function/
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Overview

- The purpose of this specifier is to **inform** the user and the compiler that a particular function **will not return control to the calling program** when it completes execution
  - informing the user helps to prevent misuse of the function
  - informing the compiler may enable it to make some code optimizations
- The _Noreturn function specifier is a hint to the compiler
  - using _Noreturn function specifier does not stop a function from returning to its caller
    - only a promise made by the programmer to the compiler to allow it some more degree of freedom to generate optimized code
  - the degree of acceptance is implementation defined
- The `exit()` function is an example of a _Noreturn function
  - once `exit()` is called, the calling function never resumes
  - as well as `abort()` function
- _Noreturn function specifier is different from the void return type
  - a typical void function does return to the calling function
  - it just does not provide an assignable value
- If a function specified with the _Noreturn function specifier violates its promise and eventually returns to its caller (by using an explicit return statement or by reaching end of function body)
  - **the behavior is undefined**
  - **MUST NOT** return from the function
- Compilers are encouraged, but **not required**, to produce warnings or errors when a _Noreturn function appears to be capable of returning to its caller.

## Using _Noreturn

- the _Noreturn keyword appears in a function declaration
- the _Noreturn specifier may appear more than once in the same function declaration
  - the behavior is the same as if it appeared once
- this specifier is typically used through the convenience macro `noreturn`
  - provided in the header file `<stdnoreturn.h>`
  - using `_Noreturn` or `noreturn` is fine and equivalent

## Example

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdnoreturn.h>

noreturn void good_noreturn();
noreturn void undefined_noreturn();

void good_noreturn()
{
    printf("Aborting...\n");
    abort();
}

void undefined_noreturn()
{
    printf("Aborting...\n will cause undefined behavior!");
}

int main()
{
    good_noreturn();
    // undefined_noreturn();
    return 0;
}
```

