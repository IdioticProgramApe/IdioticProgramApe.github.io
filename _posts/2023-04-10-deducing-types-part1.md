---
title: Deducing Types I
author: ipa
date: 2023-04-10
categories: [C-Cpp Language]
tags: [coding, modern cpp, cpp]
---

> Type duction is applicable on:
> - function templates (c++98 & c++11 & c++14, etc)
> - `auto` and `decltype` (c++11 & c++14, etc)
> - `decltype(auto)` (c++14, etc)
{: .prompt-info }

## Template Type Deduction

A general look of function template:

```c++
// declaration a function template
template<typename T>
void func(ParamType param);

// use the function template
func(expr)
```

Most commonly, **ParamType** has 3 different forms:

- **ParamType** is a pointer or reference type, but not a universal reference.
- **ParamType** is a universal reference.
- **ParamType** is neither a pointer nor a reference.

We will need **expr** to deduce **ParamType** and **T**

### ParamType: Pointer or Non-Uni Reference

> 1. If **expr**'s type is a reference, ignore the reference part
> 2. Then pattern-match **expr**'s type against **ParamType** to determine **T**
{: .prompt-tip}

With templates as such, we will have the following type deduction:

#### Reference

```c++
template<typename T>
void func1(T& param);
```

| Vairable Declaration | Function Call | **expr**                 | **T**       | **ParamType** |
| -------------------- | ------------- | ------------------------ | ----------- | ------------- |
| `int x = 27;`        | `func1(x);`   | `int`                    | `int`       | `int&`        |
| `const int cx = x;`  | `func1(cx);`  | `const int`              | `const int` | `const int&`  |
| `const int& rx = x;` | `func1(rx);`  | reference to `const int` | `const int` | `const int&`  |

#### Const Reference

```c++
template<typename T>
void func2(const T& param);
```

| Vairable Declaration | Function Call | **expr**                 | **T** | **ParamType** |
| -------------------- | ------------- | ------------------------ | ----- | ------------- |
| `int x = 27;`        | `func2(x);`   | `int`                    | `int` | `const int&`  |
| `const int cx = x;`  | `func2(cx);`  | `const int`              | `int` | `const int&`  |
| `const int& rx = x;` | `func2(rx);`  | reference to `const int` | `int` | `const int&`  |

#### Pointer

```c++
template<typename T>
void func3(T* param);
```

| Vairable Declaration  | Function Call | **expr**               | **T**       | **ParamType** |
| --------------------- | ------------- | ---------------------- | ----------- | ------------- |
| `int x = 27;`         | `func3(&x);`  | `int`                  | `int`       | `int*`        |
| `const int *px = &x;` | `func3(px);`  | pointer to `const int` | `const int` | `const int*`  |

### ParamType: Universal Reference

> 1. If **expr** is an lvalue, both **T** and **ParamType** are deduced to be lvalue references. That’s doubly unusual.
>    1. it’s the only situation in template type deduction where **T** is deduced to be a reference. 
>    2. although **ParamType** is declared using the syntax for an rvalue reference, its deduced type is an lvalue reference
> 2. If **expr** is an rvalue, the “normal reference” rules apply.
{: .prompt-tip}

```c++
template<typename T>
void func(T&& param);
```

| Vairable Declaration | Function Call | **expr**                         | **T**        | **ParamType** |
| -------------------- | ------------- | -------------------------------- | ------------ | ------------- |
| `int x = 27;`        | `func(x)`     | `int`, lvalue                    | `int&`       | `int&`        |
| `const int cx = x;`  | `func(cx)`    | `const int`, lvalue              | `const int&` | `const int&`  |
| `const int& rx = x;` | `func(rx)`    | reference to `const int`, lvalue | `const int&` | `const int&`  |
| `27`                 | `func(27)`    | magic number, rvalue             | `int`        | `int&&`       |

### ParamType: Neither Pointer Nor Reference

> 1. If **expr**'s type is a reference, ignore the reference part
> 2. If after ignoring **expr**’s reference-ness, **expr** is `const`, ignore that, too. If it’s `volatile`, also ignore that. 
{: .prompt-tip}

```c++
template<typename T>
void func(T param);  // pass by-value
```

| Vairable Declaration | Function Call | **expr**                 | **T** | **ParamType** |
| -------------------- | ------------- | ------------------------ | ----- | ------------- |
| `int x = 27;`        | `func(x)`     | `int`                    | `int` | `int`         |
| `const int cx = x;`  | `func(cx)`    | `const int`              | `int` | `int`         |
| `const int& rx = x;` | `func(rx)`    | reference to `const int` | `int` | `int`         |

### Array Arguments

Considering these 2 variables:

```c++
const char name[] = "J. P. Briggs";		// name's type is const char[13]
const char * ptrToName = name; 			// array decays to pointer
```

for the following statement:

```c++
f(name)
```

we have 2 way of deducing its actual type.

#### With By-value Parameter

```c++
template<typename T>
void f(T param);
```

Because of the *array-to-pointer decay rule*, the array declaration is treated as a pointer declaration, which means **T** will be deduced as `const char*` instead of `char[13]`

#### With By-ref Parameter

```c++
template<typename T>
void f(T& param);
```

**T** will be deduced as `const char (&)[13]` as an array instead of a pointer. This property provide us a method to get the size of an array just by type deduction **at compile-time**:

```c++
template<typename T, std::size_t N>
constexpr std::size_t arraySize(T (&)[N]) noexcept
{
    return N;
}
```

Since the value is deduced at compile-time, we can use its value to allocate another array, as:

```c++
int keyVals[] = { 1, 3, 7, 9, 11, 22, 35 }; // 7 elements
int mappedVals[arraySize(keyVals)];			// also 7 elements
std::array<int, arraySize(keyVals)> mappedVals; // contains 7 int
```

### Function Arguments

Function can be also decay into function pointer. Taking an example of:

```c++
void callback(int, double);

f(callback);	// a function takes a callback function
```

#### With By-value Parameter

```c++
template<typename T>
void f(T param);
```

**T** will be deduced as `void (*)(int, double)`

#### With By-ref Parameter

```c++
template<typename T>
void f(T& param);
```

**T** will be deduced as `void (&)(int, double)`

## `auto` Type Deduction

`auto` type deduction is template type deduction with one more exception. When a variable is declared using `auto`:

- `auto` plays the role of **T** in the template
- type specifier for the variable acts as **ParamType**
- everything on the right of `=` is considered as **expr** in the template

### General Cases

| Scenario                      | Statement             | **expr**       | `auto`       | type specifier |
| ----------------------------- | --------------------- | -------------- | ------------ | -------------- |
| Neither Pointer Nor Reference | `auto x = 27;`        | magic number   | `int`        | `int`          |
| Neither Pointer Nor Reference | `const auto cx = x;`  | `int`          | `int`        | `const int`    |
| Pointer or Non-Uni Reference  | `const auto& rx = x;` | `int`          | `int`        | `const int&`   |
| Universal Reference           | `auto&& uref1 = x;`   | `int`          | `int&`       | `int&`         |
| Universal Reference           | `auto&& uref2 = cx;`  | `const int`    | `const int&` | `const int&`   |
| Universal Reference           | `auto&& uref3 = 27;`  | `magic number` | `int`        | `int&&`        |

### Array & Function

| Scenario                                    | Statement                 | **expr**            | `auto`                  | type specifier          |
| ------------------------------------------- | ------------------------- | ------------------- | ----------------------- | ----------------------- |
| array `const char name[] = "R. N. Briggs";` | `auto arr1 = name; `      | `const char[13]`    | `const char*`           | `const char*`           |
| array `const char name[] = "R. N. Briggs";` | `auto& arr2 = name; `     | `const char[13]`    | `const char[13]`        | `const char (&)[13]`    |
| function `void someFunc(int, double);`      | `auto func1 = someFunc;`  | `void(int, double)` | `void (*)(int, double)` | `void (*)(int, double)` |
| function `void someFunc(int, double);`      | `auto& func2 = someFunc;` | `void(int, double)` | `void(int, double)`     | `void (&)(int, double)` |

### Initialization

> There are 4 way syntactic choices for initializing something in c++:
>
> | Index | Statement          | Suppoted Standard                                            |
> | ----- | ------------------ | ------------------------------------------------------------ |
> | 1     | `int x1 = 27;`     | c++98 & c++11, etc                                           |
> | 2     | `int x2(27);`      | c++98 & c++11, etc                                           |
> | 3     | `int x3 = { 27 };` | c++11, etc (using `std::initializer_list<T>`), uniform initialization |
> | 4     | `int x4{ 27 };`    | c++11, etc (using `std::initializer_list<T>`), uniform initialization |
> 
{: .prompt-info }

**Special rule** for `auto` type deduction:

When the initializer for an `auto`-declared variable is enclosed in braces, the deduced type is a `std::initializer_list`. If such a type can’t be deduced (e.g., because the values in the braced initializer are of different types), the code will be rejected. For example:

```c++
auto x5 = { 1, 2, 3.0 }; 	// error, cannot deduce the type for the initializer_list
```

There are 2 deductions happening:

1. `auto` type deduction for **x5**, which is a `std::initializer_list`, a template class
2. template type deduction for `std::initializer_list`, which led to an error.

> Things to Remember
> - `auto` type deduction is usually the same as template type deduction, but `auto` type deduction assumes that a braced initializer represents a `std::initializer_list`, and template type deduction doesn’t.
> - `auto` in a function return type or a lambda parameter implies template type deduction, not `auto` type deduction.
{: .prompt-warning }