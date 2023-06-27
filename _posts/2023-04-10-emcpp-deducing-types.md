---
title: Deducing Types
author: ipa
date: 2023-04-10
categories: [C-Cpp Language]
tags: [coding, modern cpp, cpp]
---

## Item 1: Understand template type deduction

### Snippets

- get array size at compile time, pass-by-reference template:

  ```c++
  /** 
   * return size of an array as a compile-time constant.
   */
  template<typename T, std::size_t N>
  constexpr std::size_t arraySize(T (&)[N]) noexcept
  {
      return N;
  }
  ```

### Key points

- During template type deduction, arguments that are references are treated as non-references, i.e., their reference-ness is ignored.
- When deducing types for universal reference parameters, lvalue arguments get special treatment.
- When deducing types for by-value parameters, `const` and/or `volatile` arguments are treated as **non-const** and **non-volatile**
- During template type deduction, arguments that are array or function names **decay to pointers, unless** they’re used to initialize references

## Item 2: Understand auto type deduction

### Snippets

- use auto to deduce type for braced initializer:

  ```c++
  auto x1 = 27;        // type is int, value is 27
  auto x2(27);         // ditto
  auto x3 = { 27 };    // type is std::initializer_list<int>, value is { 27 }
  auto x4{ 27 };       // ditto
  ```

### Key points

- `auto` type deduction is usually the same as `template` type deduction, but `auto` type deduction assumes that a **braced initializer** represents a `std::initializer_list`, and `template` type deduction doesn’t.
- `auto` in a function return type or a lambda parameter implies `template` type deduction, not `auto` type deduction.

## Item 3: Understand decltype

### Snippets

- get i-th value from a container:

  ```c++
  /**
   * c++ 14 version, accept deltype(auto)
   */ 
  template<typename Container, typename Index>
  decltype(auto) authAndAccess(Container&& c, Index i)
  {
      authenticateUser();
      return std::forward<Container>(c)[i];
  }
  ```

  ```c++
  /**
   * c++ 11 version, auto + trailing return type
   */ 
  template<typename Container, typename Index>
  auto authAndAccess(Container&& c, Index i) -> decltype(std::forward<Container>(c)[i])
  {
      authenticateUser();
      return std::forward<Container>(c)[i];
  }
  ```

- difference between **x** and **(x)**:

  ```c++
  int x = 0;
  
  /** decltype(x) is int, so f1 returns int */
  decltype(auto) f1() { return x; }
  /** decltype((x)) is int&, so f2 returns int& */
  decltype(auto) f1() { return (x); }
  ```

### Key points

- `decltype` almost always yields the type of a variable or expression without any modifications.
- For lvalue expressions of type **T** other than names, `decltype` always reports a type of **T&**.
- **C++14** supports `decltype(auto)`, which, like `auto`, deduces a type from its initializer, but it performs the type deduction using the `decltype` rules.

## Item 4: Know how to view deduced types

### Snippets

- using compiler error message:

  ```c++
  template<typename T> // declaration only for TD;
  class TD;
  
  TD<decltype(x)> xType; // elicit errors containing x's type
  ```

- output at runtime with **std**:

  ```c++
  /** check std::type_info::name */ 
  std::cout << typeid(x).name() << '\n'; // display types for x, output varies from compiler to compiler
  ```

- output at runtime with **boost.typeindex**:

  ```c++
  #include <boost/type_index.hpp>
  
  template<typename T>
  void f(const T& param)
  {
      using std::cout;
      using boost::typeindex::type_id_with_cvr;
      // show T
      cout << "T = "
           << type_id_with_cvr<T>().pretty_name()
           << '\n';
      // show param's type
      cout << "param = "
           << type_id_with_cvr<decltype(param)>().pretty_name()
           << '\n';
  }
  ```


### Key points

- Deduced types can often be seen using IDE editors, compiler error messages, and the Boost TypeIndex library.
- The results of some tools may be neither helpful nor accurate, so an understanding of C++’s type deduction rules remains essential.
