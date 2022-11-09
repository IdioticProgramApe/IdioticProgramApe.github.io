---
layout: default
title: Precedence
parent: Python
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
## Operator Precedence Table

| Operator                                                     | Description                                        | Notes                                                        |
| ------------------------------------------------------------ | -------------------------------------------------- | ------------------------------------------------------------ |
| `()`                                                         | Parentheses                                        |                                                              |
| `f(args...)`                                                 | Function call                                      |                                                              |
| `(expressions…)`, `[expressions…]`, `{expressions…}`, and `{key: value…}` | Displaying sequences, and dictionaries             |                                                              |
| `x[index]`, `x[index_start:index_end]`, `x(arguments)`, `x.attribute` | Indexing, slicing, calling, attribute referencing  |                                                              |
| `await x`                                                    | Await expression                                   | `asyncio` module                                             |
| `**`                                                         | Exponent                                           |                                                              |
| `+x`, `–x`, `~x`                                             | Positive, negative, bitwise NOT                    |                                                              |
| `*`, `/`, `//`, `%`                                          | Multiplication, true division, division, remainder |                                                              |
| `+`, `–`                                                     | Addition, subtraction                              |                                                              |
| `<<`, `>>`                                                   | Bitwise left and right shifts                      |                                                              |
| `&`                                                          | Bitwise AND                                        |                                                              |
| `^`                                                          | Bitwise XOR                                        | `operator` module                                            |
| `|`                                                          | Bitwise OR                                         |                                                              |
| `in`, `not in`, `is`, `is not`, `<`, `>`, `<=`, `>=`, `!=`, `==` | Comparisons, identity, and membership operators    | `a < b < c` is equivalent to `(a < b) and (b < c)`           |
| `not`                                                        | Boolean NOT                                        |                                                              |
| `and`                                                        | Boolean AND                                        |                                                              |
| `or`                                                         | Boolean OR                                         |                                                              |
| `if condition: expression_a else: expression_b`              | Conditional expression                             | ternary style: <br>`expression_a if condition else expression_b` |
| `lambda`                                                     | Lambda expression                                  |                                                              |

## TLDR

PEMDAS rules:

- **P** means **parentheses**
- **E** means **exponent**
- **M** means **multiplication**
- **D** means **division**
- **A** means **addition**
- **S** means **Subtraction**