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
- Perfect-forwarding constructors are especially problematic, because they're **typically better matches than copy constructors for non-const lvalues**, and they can hijack derived class calls to base class copy and move constructors

## Item 27: Familiarize yourself with alternatives to overloading on universal references

### Abandon overloading

- avoid the drawbacks of overloading on universal references by simply using different names for the would-be overloads

### Pass by const T&

- replace pass-by-universal-reference with pass-by-lvalue-reference-to-const as does in C++98
- this design is not very efficient

### Pass by value

- replace pass-by-reference parameters with pass by value
- consider passing objects by value when you know you'll copy them

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

- use `std::enable_if` to rachet down the conditions under which the function template that the universal reference is part of is permitted to be employed, force compilers to behave as if a particular template didn't exist

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

## Item 29: Assume that move operations are not present, not cheap, and not used

### std::array

- std::array is essentially a builtin array with an STL interface, the data for a std::array's contents are stored directly in the std::array object, which makes **the move operation for std::array has linear-time computational complexity**
- for most of other STL containers such as std::vector, they store their contents on the heap. Objects of such container types hold (as data members), conceptually, **only a pointer to the heap memory storing the contents of the container**, which can finish a move op in constant time by just copy the pointer and set the original one to nullptr.

### std::string

- std::string offers **constant-time moves** and **linear-time copies** for long strings when SSO is not applied.
- with small string optimization(**SSO**): "small" strings (e.g., those **with a capacity of no more than 15 characters**) are stored in a buffer within the std::string object; no heap-allocated storage is used

### Key points

- from c++98 to c++11, the underlying copy operations may be replaced with move operations **only if the move operations are known to not throw**
- scenarios where **move is useless**:
  - **No move operations**: The object to be moved from fails to offer move operations. The move request therefore becomes a copy request
  - **Move not faster**: The object to be moved from has move operations that are no faster than its copy operations
  - **Move not usable**: The context in which the moving would take place requires a move operation that emits no exceptions, but that operation isn't declared noexcept
  - **Source object is lvalue**: With very few exceptions (see e.g., **Item 25**) only rvalues may be used as the source of a move operation
- Assume that move operations are not present, not cheap, and not used
- In code with known types or support for move semantics, there is no need for assumptions

## Item 30: Familiarize yourself with perfect forwarding failure cases

### Perfect-forward failure cases

Given the forward function:

```c++
template<typename... Ts>
void fwd(Ts&&... params) // accept any arguments
{
    f(std::forward<Ts>(params)...); // forward them to f
}
```

here are the failure cases for the forwarding:

#### Braced initializers

```c++
void f(const std::vector<int>& v);
fwd({ 1, 2, 3 })   // compiler is forbidden from deducing a type for {...}
```

- **Compilers are unable to deduce a type for one or more of fwd's parameters**. In this case, the code fails to compile

- **Compilers deduce the "wrong" type for one or more of fwd's parameters**. Here, "wrong" could mean that fwd's instantiation won't compile with the types that were deduced, but it could also mean that the call to f using fwd's deduced types behaves differently from a direct call to f with the arguments that were passed to fwd

- **solution**:

  ```c++
  auto il = { 1, 2, 3 };
  fwd(il);
  ```

#### 0 or NULL as null pointers

- Neither 0 nor NULL can be perfect-forwarded as a null pointer
- **solution**: pass `nullptr` instead of 0 or NULL

#### Declaration-only integral static const data members

- **const propagation**: no need to define integral static const data members in classes; declarations alone suffice

  ```c++
  class Widget
  {
  public:
      static const std::size_t MinVals = 28; // MinVals' declaration
  };
  
  std::vector<int> widgetData;
  widgetData.reserve(Widget::MinVals); // no definition, ok
  ```

- If MinVals' address were to be taken (e.g., if somebody created a **pointer to MinVals**), then MinVals would require storage (so that the pointer had something to point to), the code would **fail at link-time** until a definition for MinVals was provided as the following code:

  ```c++
  fwd(Widget::MinVals);
  ```

  **References, in the code generated by compilers, are usually treated like pointers**. references are simply pointers that are automatically dereferenced

- **solution**: add the following code in the right cpp file:

  ```c++
  const std::size_t Widget::MinVals; // provide the definition, no need to give the value
  ```

#### Overloaded function names and template names

- Overloaded function names and template names make the forward function confuse about the type of the parameter, will cause an error when instantiate

- **solution**: specify the needed signature for the input arg

  ```c++
  using ProcessFuncType = int (*)(int);
  
  // processVal is an overloaded function
  ProcessFuncType processValPtr = processVal;
  fwd(processValPtr); // fine
  
  // workOnVal is a template function
  fwd(static_cast<ProcessFuncType>(workOnVal)); // also fine
  ```

#### Bitfields

- a struct using bitfields:

  ```c++
  struct IPv4Header 
  {
      std::uint32_t version:4;
      std::uint32_t IHL:4;
      std::uint32_t DSCP:6;
      std::uint32_t ECN:2;
      std::uint32_t totalLength:16;
  };
  ```

- C++ standard: **a non-const reference shall not be bound to a bit-field**, perfect forward a such non-const bitfield will lead to an error

  - C++ dictates that the **smallest thing you can point to is a char**, as there’s no way to create a pointer to arbitrary bits.

- **solution**: reference to a bit is forbidden in c++, Item 6

  ```c++
  auto length = static_cast<std::uint16_t>(h.totalLength);
  fwd(length);    // forward the copy
  ```

### Key points

- Perfect forwarding fails when **template type deduction fails** or when **it deduces the wrong type**
- The kinds of arguments that lead to perfect forwarding failure are: **braced initializers**, **null pointers expressed as 0 or NULL**, **declaration-only integral const static data members**, **template and overloaded function names**, and **bitfields**
