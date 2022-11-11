---
layout: default
title: Object Oriented Python
parent: Meta-Programming
grand_parent: Python
nav_order: 3
permalink: /docs/python/meta-programming/oop/
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>
## About Metaprogramming

### Definition

Metaprogramming is producing code that manipulates code, including:
- reads code
- modifies code
- outputs code

### Goal

Make the code **dry**, aka don't repeat the code.

### Examples

| Name      | Notes                                                        |
| --------- | ------------------------------------------------------------ |
| Quine     | A program which output itself, ref: [Quine (computing) - Wikipedia](https://en.wikipedia.org/wiki/Quine_(computing)) |
| Compilers | More advanced, compiler compiler.                            |
| Servers   | Servers builders(Django), open a webpage is downloading code that browser execute) |
| ...       | ...                                                          |

## Python Class

### Example

```python
class Foo:
    def greet(self):
        print(f"hello from {self}")
```

### Take-aways

- `class` keyword to declare a class
- `methods` is the function inside the class
- `self` is always the first parameter for regular methods
- `self` is equivalent to `this` in c++/java