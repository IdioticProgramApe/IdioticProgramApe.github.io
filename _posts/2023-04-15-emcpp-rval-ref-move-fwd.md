---
title: Rvalue References, Move Semantics and Perfect Forwarding
author: ipa
date: 2023-04-15
categories: [C-Cpp Language]
tags: [coding, modern cpp, cpp]
mermaid: true
img_path: /moderncpp/
---

## Item 23: Understand std::move and std::forward

### Snippets

- sample implementation of move

  ```c++
  // c++ 11
  template<typename T>
  typename remove_reference<T>::type&& move(T&& param)
  {
      using ReturnType = typename remove_reference<T>::type&&;
      return static_cast<ReturnType>(param);
  }
  
  // c++ 14
  template<typename T>
  decltype(auto) move(T&& param)
  {
      using ReturnType = remove_reference_t<T>&&;
      return static_cast<ReturnType>(param);
  }
  ```

### Key points

- `std::move` performs an unconditional cast to an rvalue. In and of itself, it doesn't move anything
  - don't declare objects `const` if you want to be able to move from them. Move requests on const objects are silently transformed into copy operations
- `std::forward` casts its argument to an rvalue only if that argument is bound to an rvalue
- Neither `std::move` nor `std::forward` do anything at runtime

## Item 24: Distinguish universal references from rvalue references

### Key points

- If a function template parameter has type **T&&** for a deduced type T, or if an object is declared using **auto&&**, the parameter or object is a **universal reference**
- If the form of the type declaration isn't precisely type&&, or if type deduction does not occur, type&& denotes an **rvalue reference**
- Universal references correspond to rvalue references if they're initialized with rvalues. They correspond to lvalue references if they're initialized with lvalues

## Item 25: Use std::move on rvalue references, std::forward on universal references

### Copy ellision conditions(RVO)

- the type of the local object is the same as that returned by the function
- the local object is what's being returned
- by-value function parameters don't qualify, but if they are returned, compiler must treat them as rvalues

### Key points

- Apply `std::move` to rvalue references and `std::forward` to universal references the last time each is used
- Do the same thing for rvalue references and universal references being returned from functions that return by value
- Never apply `std::move` or `std::forward` to local objects if they would otherwise be eligible for the **return value optimization**

## Item 26: Avoid overloading on universal references

### Key points

- In situations where a template instantiation and a non-template function (i.e., a "normal" function) are equally good matches for a function call, the normal function is preferred
- Overloading on universal references almost always leads to the universal reference overload being called more frequently than expected
- Perfect-forwarding constructors are especially problematic, because they’re **typically better matches than copy constructors for non-const lvalues**, and they can hijack derived class calls to base class copy and move constructors

## Item 27: Familiarize yourself with alternatives to overloading on universal references

### Abandon overloading

- avoid the drawbacks of overloading on universal references by simply using different names for the would-be overloads

### Pass by const T&

- replace pass-by-universal-reference with pass-by-lvalue-reference-to-const as does in C++98
- this design is not very efficient

### Pass by value

- replace pass-by-reference parameters with pass by value
- consider passing objects by value when you know you’ll copy them

### Use Tag dispatch

- let the universal reference is part of a parameter list containing other parameters that are not universal references

  ```c++
  template<typename T>
  void logAndAdd(T&& name)
  {
      logAndAddImpl(
          std::forward<T>(name), 
          std::is_integral<typename std::remove_reference<T>::type>()
      );
  }
  
  // for non-integral argument
  template<typename T> 
  void logAndAddImpl(T&& name, std::false_type)
  {
      auto now = std::chrono::system_clock::now();
      log(now, "logAndAdd");
      names.emplace(std::forward<T>(name));
  }
  
  std::string nameFromIdx(int idx);	// function declaration
  // for integral argument
  void logAndAddImpl(int idx, std::true_type)
  {
      logAndAdd(nameFromIdx(idx));
  }
  ```

### Constraining templates that take universal references

- use `std::enable_if` to rachet down the conditions under which the function template that the universal reference is part of is permitted to be employed, force compilers to behave as if a particular template didn’t exist

  ```c++
  class Person 
  {
  public:
      template<
          typename T,
          typename = std::enable_if_t<
              !std::is_base_of<Person, std::decay_t<T>>::value &&
              !std::is_integral<std::remove_reference_t<T>>::value
          >
      >
      explicit Person(T&& n) : name(std::forward<T>(n))
      {
          // assert that a std::string can be created from a T object
          static_assert(
              std::is_constructible<std::string, T>::value,
              "Parameter n can't be used to construct a std::string"
          );
      }
      
      explicit Person(int idx) : name(nameFromIdx(idx)) { ... }
      
  private:
      std::string name;
  };
  ```

### Key points

- Alternatives to the combination of universal references and overloading include **the use of distinct function names**, **passing parameters by lvalue-reference-to-const**, **passing parameters by value**, and **using tag dispatch**
- Constraining templates via `std::enable_if` permits the use of universal references and overloading together, but it controls the conditions under which compilers may use the universal reference overloads
- Universal reference parameters often have efficiency advantages, but they typically have usability disadvantages

## Item 28: Understand reference collapsing

### Snippets

- std::forward simple implementation:

  ```c++
  template<typename T>
  T&& forward(typename remove_reference<T>::type& param)
  {
      return static_cast<T&&>(param);
  }
  ```

### Key points

- Reference collapsing occurs in four contexts: **template instantiation**, **auto type generation**, **creation and use of typedefs and alias declarations**, and **decltype**
- When compilers generate a reference to a reference in a reference collapsing context, the result becomes a single reference: 
  - If either of the original references is an lvalue reference, the result is an lvalue reference
  - Otherwise it's an rvalue reference
- Universal references are rvalue references in contexts where **type deduction distinguishes lvalues from rvalues** and where **reference collapsing occurs**



