---
title: Python's Magic Methods
author: ipa
date: 2022-11-15
categories: [Python Language]
tags: [coding, python]
---

## Construction & Initialization

| Method                      | Description                                                  |
| --------------------------- | ------------------------------------------------------------ |
| `__new__(cls, [args...])`   | The first method to get called in an object's instantiation  |
| `__init__(self, [args...])` | The initializer for the class                                |
| `__del__(self)`             | The destructor (when the object is GCed), different from `del` |

## Operators

### Comparison magic methods

| Method                | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| `__eq__(self, other)` | Defines behavior for the equality operator, `==`             |
| `__ne__(self, other)` | Defines behavior for the inequality operator, `!=`           |
| `__lt__(self, other)` | Defines behavior for the less-than operator, `<`             |
| `__gt__(self, other)` | Defines behavior for the greater-than operator, `>`          |
| `__le__(self, other)` | Defines behavior for the less-than-or-equal-to operator, `<=` |
| `__ge__(self, other)` | Defines behavior for the greater-than-or-equal-to operator, `>=` |

### Numeric magic methods

#### Unary operators & functions

| Method               | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| `__pos__(self)`      | Implements behavior for unary positive (e.g. `+some_object`) |
| `__neg__(self)`      | Implements behavior for negation (e.g. `-some_object`)       |
| `__abs__(self)`      | Implements behavior for the built in `abs()` function        |
| `__invert__(self)`   | Implements behavior for inversion using the `~` operator     |
| `__round__(self, n)` | Implements behavior for the built in `round()` function. `n` is the number of decimal places to round to |
| `__floor__(self)`    | Implements behavior for `math.floor()`                       |
| `__ceil__(self)`     | Implements behavior for `math.ceil()`                        |
| `__trunc__(self)`    | Implements behavior for `math.trunc()`                       |

#### Normal arithmetic operators

| Method                      | Description                                                  |
| --------------------------- | ------------------------------------------------------------ |
| `__add__(self, other)`      | Implements addition                                          |
| `__sub__(self, other)`      | Implements subtraction                                       |
| `__mul__(self, other)`      | Implements multiplication                                    |
| `__floordiv__(self, other)` | Implements integer division using the `//` operator          |
| `__truediv__(self, other)`  | Implements division using the `/` operator. always the case in python3 |
| `__mod__(self, other)`      | Implements modulo using the `%` operator                     |
| `__divmod__(self, other)`   | Implements behavior for long division using the `divmod()` built in function |
| `__pow__`                   | Implements behavior for exponents using the `**` operator    |
| `__lshift__(self, other)`   | Implements left bitwise shift using the `<<` operator        |
| `__rshift__(self, other)`   | Implements right bitwise shift using the `>>` operator       |
| `__and__(self, other)`      | Implements bitwise and using the `&` operator                |
| `__or__(self, other)`       | Implements bitwise or using the `|` operator                 |
| `__xor__(self, other)`      | Implements bitwise xor using the `^` operator                |

#### Reflected arithmetic operators

> Note:
>
> the object on the left hand side of the operator (`other` in the example) must not define (or return `NotImplemented`) for its definition of the non-reflected version of an operation. For instance, in the example, `some_object.__radd__` will only be called if `other` does not define `__add__`.
{: .prompt-tip }

| Method                       | Description                                                  |
| ---------------------------- | ------------------------------------------------------------ |
| `__radd__(self, other)`      | Implements reflected addition                                |
| `__rsub__(self, other)`      | Implements reflected subtraction                             |
| `__rmul__(self, other)`      | Implements reflected multiplication                          |
| `__rfloordiv__(self, other)` | Implements reflected integer division using the `//` operator |
| `__rtruediv__(self, other)`  | Implements reflected *true* division. always the case in python3 |
| `__rmod__(self, other)`      | Implements reflected modulo using the `%` operator           |
| `__rdivmod__(self, other)`   | Implements behavior for long division using the `divmod()` built in function, when `divmod(other, self)` is called |
| `__rpow__`                   | Implements behavior for reflected exponents using the `**` operator |
| `__rlshift__(self, other)`   | Implements reflected left bitwise shift using the `<<` operator |
| `__rrshift__(self, other)`   | Implements reflected right bitwise shift using the `>>` operator |
| `__rand__(self, other)`      | Implements reflected bitwise and using the `&` operator      |
| `__ror__(self, other)`       | Implements reflected bitwise or using the `|` operator       |
| `__rxor__(self, other)`      | Implements reflected bitwise xor using the `^` operator      |

#### Augmented assignment

| Method                       | Description                                                  |
| ---------------------------- | ------------------------------------------------------------ |
| `__iadd__(self, other)`      | Implements addition with assignment, `+=`                    |
| `__isub__(self, other)`      | Implements subtraction with assignment, `-=`                 |
| `__imul__(self, other)`      | Implements multiplication with assignment, `*=`              |
| `__ifloordiv__(self, other)` | Implements integer division with assignment using the `//=` operator |
| `__itruediv__(self, other)`  | Implements division with assignment using the `/=` operator. always the case in python3 |
| `__imod__(self, other)`      | Implements modulo with assignment using the `%=` operator    |
| `__ipow__`                   | Implements behavior for exponents with assignment using the `**=` operator |
| `__ilshift__(self, other)`   | Implements left bitwise shift with assignment using the `<<=` operator |
| `__irshift__(self, other)`   | Implements right bitwise shift with assignment using the `>>=` operator |
| `__iand__(self, other)`      | Implements bitwise and with assignment using the `&=` operator |
| `__ior__(self, other)`       | Implements bitwise or with assignment using the `|=` operator |
| `__ixor__(self, other)`      | Implements bitwise xor with assignment using the `^=` operator |

#### Type conversion magic methods

| Method              | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `__int__(self)`     | Implements type conversion to **int**                        |
| `__long__(self)`    | Implements type conversion to **long**                       |
| `__float__(self)`   | Implements type conversion to **float**                      |
| `__complex__(self)` | Implements type conversion to **complex**                    |
| `__oct__(self)`     | Implements type conversion to **octal**                      |
| `__hex__(self)`     | Implements type conversion to **hexadecimal**                |
| `__index__(self)`   | Implements type conversion to an **int** when the object is used **in a slice expression**.<br>If you define a custom numeric type that might be used in slicing, you should define `__index__` |
| `__trunc__(self)`   | Called when `math.trunc(self)` is called.<br>`__trunc__` should return the value of `self` truncated to an integral type (usually a **long**) |

## Class Representation

| Method                        | Description                                                  |
| ----------------------------- | ------------------------------------------------------------ |
| `__str__(self)`               | Defines behavior for when `str()` is called on an instance of your class, intended to be human-readable |
| `__repr__(self)`              | Defines behavior for when `repr()` is called on an instance of your class, mostly machine-readable |
| `__bytes__(self)`             | Defines behavior for when `bytes()` is called on an instance of your class |
| `__format__(self, formatstr)` | Defines behavior for when an instance of your class is used in new-style string formatting |
| `__hash__(self)`              | Defines behavior for when `hash()` is called on an instance of your class, often used together with `__eq__` |
| `__bool__(self)`              | Defines behavior for when `bool()` is called on an instance of your class, Should return `True` or `False` |
| `__dir__(self)`               | Defines behavior for when `dir()` is called on an instance of your class |
| `__sizeof__(self)`            | Defines behavior for when `sys.getsizeof()` is called on an instance of your class |

## Controlling Attribute Access

| Method                           | Description                                                  |
| -------------------------------- | ------------------------------------------------------------ |
| `__getattr__(self, name)`        | Define behavior for when a user attempts to access an attribute that doesn't exist (either at all or yet) |
| `__setattr__(self, name, value)` | Define behavior for assignment to an attribute regardless of whether or not that attribute exists |
| `__delattr__(self, name)`        | Define behavior for deleting attributes, pay attention to the infinite recursion |
| `__getattribute__(self, name)`   | (**Not recommended**)Define rules for whenever an attribute's value is accessed |

## Making Custom Sequences

### Container magic methods

| Method                          | Description                                                  |
| ------------------------------- | ------------------------------------------------------------ |
| `__len__(self)`                 | Returns the length of the container                          |
| `__getitem__(self, key)`        | Defines behavior for when an item is accessed, using the notation `self[key]`<br>`TypeError` if the type of the **key is wrong** and `KeyError` if there is **no corresponding value** for the key |
| `__setitem__(self, key, value)` | (**mutable container**)Defines behavior for when an item is assigned to, using the notation `self[nkey] = value`, |
| `__delitem__(self, key)`        | (**mutable container**)Defines behavior for when an item is deleted (e.g. `del self[key]`) |
| `__iter__(self)`                | Should return an iterator for the container                  |
| `__reversed__(self)`            | (**ordered container**)Called to implement behavior for the `reversed()` built in function |
| `__contains__(self, item)`      | `__contains__` defines behavior for membership tests using `in` and `not in` |
| `__missing__(self, key)`        | `__missing__` is used in subclasses of `dict`. It defines behavior for whenever a key is accessed that does not exist in a dictionary |

## Reflection

| Method                              | Description                                                  |
| ----------------------------------- | ------------------------------------------------------------ |
| `__instancecheck__(self, instance)` | Checks if an instance is an instance of the class you defined (e.g. `isinstance(instance, class)`) |
| `__subclasscheck__(self, subclass)` | Checks if a class subclasses the class you defined (e.g. `issubclass(subclass, class)`) |

## Callable Objects

| Method                      | Description                                              |
| --------------------------- | -------------------------------------------------------- |
| `__call__(self, [args...])` | Allows an instance of a class to be called as a function |

## Context Managers

| Method                                                       | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `__enter__(self)`                                            | Defines what the context manager should do at the beginning of the block created by the `with` statement<br>the return value of `__enter__` is bound to the *target* of the `with` statement, or the name after the `as` |
| `__exit__(self, exception_type, exception_value, traceback)` | Defines what the context manager should do after its block has been executed (or terminates) |

## Building Descriptor Objects

| Method                           | Description                                                  |
| -------------------------------- | ------------------------------------------------------------ |
| `__get__(self, instance, owner)` | Define behavior for when the descriptor's value is retrieved.<br>`instance` is the instance of the owner class. `owner` is the owner class itself |
| `__set__(self, instance, value)` | Define behavior for when the descriptor's value is changed.<br>`instance` is the instance of the owner class and `value` is the value to set the descriptor to |
| `__delete__(self, instance)`     | Define behavior for when the descriptor's value is deleted.<br>`instance` is the instance of the owner class |

## Copying

| Method                            | Description                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| `__copy__(self)`                  | Defines behavior for `copy.copy()` for instances of your class. <br>`copy.copy()` returns a ***shallow copy*** of your object, All the values are the reference despite the object is new |
| `__deepcopy__(self, memodict={})` | Defines behavior for `copy.deepcopy()` for instances of your class.<br>`copy.deepcopy()` returns a ***deep copy*** of your object -- the object *and* its data are both copied |

## Data Serialization/Pickling

| Method                      | Description                                                  |
| --------------------------- | ------------------------------------------------------------ |
| `__getinitargs__(self)`     | (**old-style class**)Define behavior for `__init__` to be called when the class is unpickled.<br>return a tuple of the arguments that need to be passed to `__init__` |
| `__getnewargs__(self)`      | (**new-style class**)Define behavior influence what arguments get passed to `__new__` upon unpickling.<br>return a tuple of arguments that will then be passed to `__new__` |
| `__getstate__(self)`        | Define behavior for returning a custom state to be stored when the object is pickled<br>Called through `pickle.dump(pkl_file, obj)` |
| `__setstate__(self, state)` | Define behavior for the object's state being passed to it instead of directly applied to the object's `__dict__`<br>Called through `data = pickle.load(pkl_file)` |
| `__reduce__(self)`          | Called when an object defining it is pickled<br>Either return a string representing a global name that Python will look up and pickle, or a tuple |
| `__reduce_ex__(self)`       | Exists for compatibility                                     |

(For more details, please check the orginal post: [A Guide to Python's Magic Methods](https://rszalski.github.io/magicmethods/))

