---
title: Data Type Range
author: ipa
date: 2023-08-14
categories: [Miscellaneous]
tags: [definitions]
math: true
---

## Standard Types

| Type Name            | Bytes  | Other Names                     | Range of Values                                              | Notes                                          |
| -------------------- | ------ | ------------------------------- | ------------------------------------------------------------ | ---------------------------------------------- |
| `int`                | 4      | `signed`                        | $`\left\{ n \vert -2 147 483 648 \leqslant n \leqslant 2 147 483 647, n \in \mathbb{Z} \right\}`$ |                                                |
| `unsigned int`       | 4      | `unsigned`                      | $`\left\{ n \vert 0 \leqslant n \leqslant 4 294 967 295, n \in \mathbb{Z} \right\}`$ |                                                |
| `bool`               | 1      | -                               | `false` or `true`                                            |                                                |
| `char`               | 1      | -                               | $`\left\{ n \vert -128 \leqslant n \leqslant 127, n \in \mathbb{Z} \right\}`$ |                                                |
| `signed char`        | 1      | -                               | $`\left\{ n \vert -128 \leqslant n \leqslant 127, n \in \mathbb{Z} \right\}`$ |                                                |
| `unsigned char`      | 1      | -                               | $`\left\{ n \vert 0 \leqslant n \leqslant 255, n \in \mathbb{Z} \right\}`$ |                                                |
| `short`              | 2      | `short int`, `signed short int` | $`\left\{ n \vert -32 768 \leqslant n \leqslant 32 767, n \in \mathbb{Z} \right\}`$ |                                                |
| `unsigned short`     | 2      | `unsigned short int`            | $`\left\{ n \vert 0 \leqslant n \leqslant 65 535, n \in \mathbb{Z} \right\}`$ |                                                |
| `long`               | 4      | `long int`, `signed long int`   | $`\left\{ n \vert -2 147 483 648 \leqslant n \leqslant 2 147 483 647, n \in \mathbb{Z} \right\}`$ |                                                |
| `unsigned long`      | 4      | `unsigned long int`             | $`\left\{ n \vert 0 \leqslant n \leqslant 4 294 967 295, n \in \mathbb{Z} \right\}`$ |                                                |
| `long long`          | 8      | -                               | $`\left\{ n \vert -9 223 372 036 854 775 808 \leqslant n \leqslant 9 223 372 036 854 775 807, n \in \mathbb{Z} \right\}`$ |                                                |
| `unsigned long long` | 8      | -                               | $`\left\{ n \vert 0 \leqslant n \leqslant 18 446 744 073 709 551 615, n \in \mathbb{Z} \right\}`$ |                                                |
| `enum`               | varies | -                               | -                                                            |                                                |
| `float`              | 4      | -                               | $`[-3.4\times10^{38}, -1.18\times10^{-38}] \cup [1.18\times10^{-38}, 3.4\times10^{38}] \cup { 0.0  \right\}`$ | **6-9** significant digits, typically **7**    |
| `double`             | 8      | -                               | $`[-1.80\times10^{308}, -2.23\times10^{-308}] \cup [2.23\times10^{-308}, 1.80\times10^{308}] \cup \left\{ 0.0 \right\}`$ | **15-18** significant digits, typically **16** |
| `long double`        | 8      | -                               | $`[-1.80\times10^{308}, -2.23\times10^{-308}] \cup [2.23\times10^{-308}, 1.80\times10^{308}] \cup \left\{ 0.0 \right\}`$ | same as `double`                               |
| `wchar_t`            | 2      | -                               | $`\left\{ n \vert 0 \leqslant n \leqslant 65 535, n \in \mathbb{Z} \right\}`$ |                                                |

## Microsoft C++ Compiler

| Type Name          | Bytes | Other Names                              | Range of Values                                              | Notes |
| ------------------ | ----- | ---------------------------------------- | ------------------------------------------------------------ | ----- |
| `__int8`           | 1     | `char`                                   | $`\left\{ n \vert -128 \leqslant n \leqslant 127, n \in \mathbb{Z} \right\}`$ |       |
| `unsigned __int8`  | 1     | `unsigned char`                          | $`\left\{ n \vert 0 \leqslant n \leqslant 255, n \in \mathbb{Z} \right\}`$ |       |
| `__int16`          | 2     | `short`, `short int`, `signed short int` | $`\left\{ n \vert -32 768 \leqslant n \leqslant 32 767, n \in \mathbb{Z} \right\}`$ |       |
| `unsigned __int16` | 2     | `unsigned short`, `unsigned short int`   | $`\left\{ n \vert 0 \leqslant n \leqslant 65 535, n \in \mathbb{Z} \right\}`$ |       |
| `__int32`          | 4     | `signed`, `signed int`, `int`            | $`\left\{ n \vert -2 147 483 648 \leqslant n \leqslant 2 147 483 647, n \in \mathbb{Z} \right\}`$ |       |
| `unsigned __int32` | 4     | `unsigned`, `unsigned int`               | $`\left\{ n \vert 0 \leqslant n \leqslant 4 294 967 295, n \in \mathbb{Z} \right\}`$ |       |
| `__int64`          | 8     | `long long`, `signed long long`          | $`\left\{ n \vert -9 223 372 036 854 775 808 \leqslant n \leqslant 9 223 372 036 854 775 807, n \in \mathbb{Z} \right\}`$ |       |
| `unsigned __int64` | 8     | `unsigned long long`                     | $`\left\{ n \vert 0 \leqslant n \leqslant 18 446 744 073 709 551 615, n \in \mathbb{Z} \right\}`$ |       |

source: [Data Type Ranges \| Microsoft Learn](https://learn.microsoft.com/en-us/cpp/cpp/data-type-ranges?view=msvc-170)
