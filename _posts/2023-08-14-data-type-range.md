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
| `int`                | 4      | `signed`                        | $$ \{ n \vert -2 147 483 648 \leqslant n \leqslant 2 147 483 647, n \in \mathbb{Z} \} $$ |                                                |
| `unsigned int`       | 4      | `unsigned`                      | $$ \{ n \vert 0 \leqslant n \leqslant 4 294 967 295, n \in \mathbb{Z} \} $$ |                                                |
| `bool`               | 1      | -                               | `false` or `true`                                            |                                                |
| `char`               | 1      | -                               | $$ \{ n \vert -128 \leqslant n \leqslant 127, n \in \mathbb{Z} \} $$ |                                                |
| `signed char`        | 1      | -                               | $$ \{ n \vert -128 \leqslant n \leqslant 127, n \in \mathbb{Z} \} $$ |                                                |
| `unsigned char`      | 1      | -                               | $$ \{ n \vert 0 \leqslant n \leqslant 255, n \in \mathbb{Z} \} $$ |                                                |
| `short`              | 2      | `short int`, `signed short int` | $$ \{ n \vert -32 768 \leqslant n \leqslant 32 767, n \in \mathbb{Z} \} $$ |                                                |
| `unsigned short`     | 2      | `unsigned short int`            | $$ \{ n \vert 0 \leqslant n \leqslant 65 535, n \in \mathbb{Z} \} $$ |                                                |
| `long`               | 4      | `long int`, `signed long int`   | $$ \{ n \vert -2 147 483 648 \leqslant n \leqslant 2 147 483 647, n \in \mathbb{Z} \} $$ |                                                |
| `unsigned long`      | 4      | `unsigned long int`             | $$ \{ n \vert 0 \leqslant n \leqslant 4 294 967 295, n \in \mathbb{Z} \} $$ |                                                |
| `long long`          | 8      | -                               | $$ \{ n \vert -9 223 372 036 854 775 808 \leqslant n \leqslant 9 223 372 036 854 775 807, n \in \mathbb{Z} \} $$ |                                                |
| `unsigned long long` | 8      | -                               | $$ \{ n \vert 0 \leqslant n \leqslant 18 446 744 073 709 551 615, n \in \mathbb{Z} \} $$ |                                                |
| `enum`               | varies | -                               | -                                                            |                                                |
| `float`              | 4      | -                               | $$ [-3.4\times10^{38}, -1.18\times10^{-38}] \cup [1.18\times10^{-38}, 3.4\times10^{38}] \cup \{ 0.0 \} $$ | **6-9** significant digits, typically **7**    |
| `double`             | 8      | -                               | $$ [-1.80\times10^{308}, -2.23\times10^{-308}] \cup [2.23\times10^{-308}, 1.80\times10^{308}] \cup \{ 0.0 \} $$ | **15-18** significant digits, typically **16** |
| `long double`        | 8      | -                               | $$ [-1.80\times10^{308}, -2.23\times10^{-308}] \cup [2.23\times10^{-308}, 1.80\times10^{308}] \cup \{ 0.0 \} $$ | same as `double`                               |
| `wchar_t`            | 2      | -                               | $$ \{ n \vert 0 \leqslant n \leqslant 65 535, n \in \mathbb{Z} \} $$ |                                                |

## Microsoft C++ Compiler

| Type Name          | Bytes | Other Names                              | Range of Values                                              | Notes |
| ------------------ | ----- | ---------------------------------------- | ------------------------------------------------------------ | ----- |
| `__int8`           | 1     | `char`                                   | $$ \{ n \vert -128 \leqslant n \leqslant 127, n \in \mathbb{Z} \} $$ |       |
| `unsigned __int8`  | 1     | `unsigned char`                          | $$ \{ n \vert 0 \leqslant n \leqslant 255, n \in \mathbb{Z} \} $$ |       |
| `__int16`          | 2     | `short`, `short int`, `signed short int` | $$ \{ n \vert -32 768 \leqslant n \leqslant 32 767, n \in \mathbb{Z} \} $$ |       |
| `unsigned __int16` | 2     | `unsigned short`, `unsigned short int`   | $$ \{ n \vert 0 \leqslant n \leqslant 65 535, n \in \mathbb{Z} \} $$ |       |
| `__int32`          | 4     | `signed`, `signed int`, `int`            | $$ \{ n \vert -2 147 483 648 \leqslant n \leqslant 2 147 483 647, n \in \mathbb{Z} \} $$ |       |
| `unsigned __int32` | 4     | `unsigned`, `unsigned int`               | $$ \{ n \vert 0 \leqslant n \leqslant 4 294 967 295, n \in \mathbb{Z} \} $$ |       |
| `__int64`          | 8     | `long long`, `signed long long`          | $$ \{ n \vert -9 223 372 036 854 775 808 \leqslant n \leqslant 9 223 372 036 854 775 807, n \in \mathbb{Z} \} $$ |       |
| `unsigned __int64` | 8     | `unsigned long long`                     | $$ \{ n \vert 0 \leqslant n \leqslant 18 446 744 073 709 551 615, n \in \mathbb{Z} \} $$ |       |

source: [Data Type Ranges \| Microsoft Learn](https://learn.microsoft.com/en-us/cpp/cpp/data-type-ranges?view=msvc-170)
