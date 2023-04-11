---
title: Deducing Types II
author: ipa
date: 2023-04-11
categories: [Cpp Language]
tags: [coding, modern cpp, cpp]
---

> Type duction is applicable on:
> - function templates (c++98 & c++11 & c++14, etc)
> - `auto` and `decltype` (c++11 & c++14, etc)
> - `decltyoe(auto)` (c++14, etc)
{: .prompt-info }

## Decltype

Given a name or an expression, `decltype` returns the name's or the expression's type. Here are some use cases for `decltype`:

- declaring function templates where the function's return type depends on its parameter types (c++11 primary use)
- placeholder

### Determine Return Type

This example takes the std container implementation:

- for most std containers, `operator[]` returns a reference to its element type, as **T&**
- exception: `std::vector<bool>`'s returns a new object, as `bool`

We want a template function to get exactly whatever type `operator[]` returns if we provide a index of it.

```c++
// c++11 version of authAndAccess
template<typename Container, typename Index>
auto authAndAccess(Container&& c, Index i) -> decltype(std::forward<Container>(c)[i])
{
    authenticateUser();
    return std::forward<Container>(c)[i];
}
```
```c++
// c++14 version of authAndAccess
template<typename Container, typename Index>
decltype(auto) authAndAccess(Container&& c, Index i)
{
	authenticateUser();
	return std::forward<Container>(c)[i];
}
```

> C++ Standard Differences:
>
> - C++11 permits return types for single-statement lambdas to be deduced, will need a *trailing return type* syntax
> - C++14 extends this to both all lambdas and all functions, including those with multiple statements. `decltype(auto)` is also allowed.
{: .prompt-info }

`auto` here in the template function **authAndAccess** means the return type will be deduced from its implementation.

### Exceptions

Applying `decltype` to a name yields the declared type for that name. `decltype` ensures that the type reported is always an lvalue reference. If an lvalue expression has type **T**, the reported type is **T&** (functions returning lvalues always return lvalue references).

Here is a difference between lvalue name and a lvalue expression:

| Variable Declaration | Name/Expression   | Reported Type | Note              |
| -------------------- | ----------------- | ------------- | ----------------- |
| `int x = 0;`         | `decltype( x )`   | `int`         | lvalue name       |
| `int x = 0;`         | `decltype( (x) )` | `int&`        | lvalue expression |

> Things to Remember
> - `decltype` almost always yields the type of a variable or expression without any modifications.
> - For lvalue expressions of type **T** other than names, `decltype` always reports a type of **T&**.
> - C++14 supports `decltype(auto)`, which, like `auto`, deduces a type from its initializer, but it performs the type deduction using the `decltype` rules.
{: .prompt-warning }
