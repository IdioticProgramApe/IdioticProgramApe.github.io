---
title: Precedence
author: ipa
date: 2022-11-09
categories: [Python Language]
tags: [coding, python, fundamental]
---

## TLDR

Parentheses `()` conquers all! to be more specific, the precedence follows PEMDAS rules:

- **P** means **parentheses**
- **E** means **exponent**
- **M** means **multiplication**
- **D** means **division**
- **A** means **addition**
- **S** means **Subtraction**

## Operator Precedence Table

| Operator                                                     | Description                                        | Notes                                                        |
| ------------------------------------------------------------ | -------------------------------------------------- | ------------------------------------------------------------ |
| `()`                                                         | Parentheses                                        |                                                              |
| `f(args...)`                                                 | Function call                                      |                                                              |
| `(expressions…)`, `[expressions…]`, `{expressions…}`, and `{key: value…}` | Displaying sequences, and dictionaries             |                                                              |
| `x[index]`, `x[index_start:index_end]`, `x(arguments)`, `x.attribute` | Indexing, slicing, calling, attribute referencing  |                                                              |
| `await x`                                                    | Await expression                                   | `asyncio` module                                             |
| `**`                                                         | Exponent                                           | (alternative) `operator` module                              |
| `+x`, `–x`, `~x`                                             | Positive, negative, bitwise NOT                    | (alternative) `operator` module                              |
| `*`, `/`, `//`, `%`                                          | Multiplication, true division, division, remainder | (alternative) `operator` module                              |
| `+`, `–`                                                     | Addition, subtraction                              | (alternative) `operator` module                              |
| `<<`, `>>`                                                   | Bitwise left and right shifts                      | (alternative) `operator` module                              |
| `&`                                                          | Bitwise AND                                        | (alternative) `operator` module                              |
| `^`                                                          | Bitwise XOR                                        | (alternative) `operator` module                              |
| `|`                                                          | Bitwise OR                                         | (alternative) `operator` module                              |
| `in`, `not in`, `is`, `is not`, `<`, `>`, `<=`, `>=`, `!=`, `==` | Comparisons, identity, and membership operators    | `a < b < c` is equivalent to `(a < b) and (b < c)`           |
| `not`                                                        | Boolean NOT                                        |                                                              |
| `and`                                                        | Boolean AND                                        |                                                              |
| `or`                                                         | Boolean OR                                         |                                                              |
| `if condition: expression_a else: expression_b`              | Conditional expression                             | ternary style: <br>`expression_a if condition else expression_b` |
| `lambda`                                                     | Lambda expression                                  |                                                              |
