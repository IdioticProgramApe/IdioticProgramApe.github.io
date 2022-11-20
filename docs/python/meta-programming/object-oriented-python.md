---
layout: default
title: Object Oriented Python
parent: Meta-Programming
grand_parent: Python
nav_order: 1
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

### Class Definition

```python
class Foo:
    def greet(self):
        print(f"hello from {self}")
```

- `class` keyword to declare a class
- `methods` is the function inside the class
- `self` is always the first parameter for regular methods, thus for calling a class method, there are 2 ways, say `foo` is an instance of `Foo`:
  - `foo.greet()`, `self` will be automatically replaced by the `foo` as the first parameter if method is called from an instance
  - `Foo.greet(foo)`, method gets called from the class `Foo`, but it needs to know which instance it works on, so we need to pass `foo` in
- `self` is equivalent to `this` in c++/java

### Special Methods

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def __str__(self):
        return str(self.name) + ', ' + str(self.age) + ' years old'
```

- `__init__` plays the constructor's most work in python class instantiation, it will take in parameters after `self`, usually its job is to save parameters in `self`, those variables are called `fields`

- `__str__` is quite useful when printing the `str` version of object, in this case `Person` instances

- there will be other methods like these 2 for objects, they are called `dunder/magic methods`, can use `dir` function to check all the dunder methods a class supports like `dir(Person)`

  - `dunder` stands for `double underscore`

  - | Python Method  | Python Syntax    |
    | -------------- | ---------------- |
    | `__eq__`       | `==`             |
    | `__cmp__`      | `>, <, ==`       |
    | `__str__`      | `str(...)`       |
    | `__contains__` | `in`             |
    | `__add__`      | `+`              |
    | `__new__`      | more complicated |

- **no** modifiers as `public`, `static`, `void`, **almost** everything is non-statics (decorator `staticmethod`)

- starts name with `_` to mark **private semantically** 

  - consider this thread for a great answer: [python - Why do some functions have underscores "__" before and after the function name? - Stack Overflow](https://stackoverflow.com/questions/8689964/why-do-some-functions-have-underscores-before-and-after-the-function-name/8689983#8689983)

### `__init__` vs `__new__`

```python
class Test:
    def __new__(cls, x):
        print(f"__new__, cls={cls}")
        return super().__new__(cls)
    
    def __init__(self, x):
        print(f"_init__, self={self}")
        self.x = x
        
test = Test(2)
```

They share the same input parameters except the first one

- `__init__` is used to initialize the instance, makes`self`
  - sets up `fields`
  - non-static, and no return
- `__new__` is used to create that instance in the first place, populates `self`
  - static, and return
  - takes in `cls` instead of `self`
  - if superclass's `__new__` is called, `__init__` will also be called

### Everything is an Object

Use `type` function and `.mro()` method to verify this statement, `mro` stands for `method resolution order` which reveals the inheritance hierarchy

### Duck Typing

`"If it walks like a duck and it quacks like a duck, then it must be a duck" -- James Whitcomb Riley` ([Duck typing - Wikipedia](https://en.wikipedia.org/wiki/Duck_typing))

- The official type of an object is regardless, only how this object behaves matters
- Python is a dynamic programming language, not a strong typed language as C/C++, java or rus
  - still recommend use typing module to type the scripts
- `__dict__`, a reflection mechanism, the ability to see the code we are executing currently as data in that code.

