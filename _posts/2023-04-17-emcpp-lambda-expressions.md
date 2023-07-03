---
title: Lambda Expressions
author: ipa
date: 2023-04-17
categories: [C-Cpp Language]
tags: [coding, modern cpp, cpp]
img_path: /moderncpp/
---

## Item 31: Avoid default capture modes

### Key points

- code in the body of the lambda is in the scope of the closure class  
- lambda captures apply only to **non-static local variables (including parameters)** visible in the scope where the lambda is created
- Default by-reference capture aka **[&]** can lead to dangling references.
- Default by-value capture aka **[=]** is susceptible to dangling pointers (especially `this`), and it misleadingly suggests that lambdas are self-contained

## Item 32: Use init capture to move objects into closures

### Init capture

- another name: **generalized lambda capture**
- specify **the name of a data member** in the closure class generated from the lambda
- specify **an expression** initializing that data member
- using the form of `[lfs = rhs]`:
  - **lhs**: the name of the data member in the **closure class**
  - **rhs**: the initializing expression

### Snippets

- move std::unique_ptr into a closure in **c++14**:

  ```c++
  class Widget 
  {
  public:
      bool isValidated() const;
      bool isProcessed() const;
      bool isArchived() const;
  };
  
  auto pw = std::make_unique<Widget>();
  
  // create a data member pw in the closure, and initialize that data member with 
  // the result of applying std::move to the local variable pw.
  auto func = [pw = std::move(pw)]{ return pw->isValidated() && pw->isArchived(); };
  ```

- move std::unique_ptr into a closure in **c++11**, equivalent to a functor:

  ```c++
  class IsValAndArch 
  {
  public:
      using DataType = std::unique_ptr<Widget>;
      
      explicit IsValAndArch(DataType&& ptr) : pw(std::move(ptr)) {}
      
      bool operator()() const { return pw->isValidated() && pw->isArchived(); }
  private:
      DataType pw;
  };
  
  auto func = IsValAndArch(std::make_unique<Widget>());
  ```

- or use std::bind in **c++11**, which returns a bind object which contains copies of all the arguments passed to `std::bind`:

  - For each **lvalue** argument, the corresponding object in the bind object is **copy constructed**

  - For each **rvalue**, it’s **move constructed**


  ```c++
  std::vector<double> data;
  
  auto func = std::bind([](const std::vector<double>& data)
      {
          /* uses of data */ 
      },
      std::move(data)
  );
  ```

- mutable lambda in **c++11**:

  ```c++
  auto func = std::bind([](std::vector<double>& data) mutable
      { 
          /* uses of data */ 
      },
      std::move(data)
  );
  ```

### Key points

- Use C++14's init capture to move objects into closures
- In C++11, emulate init capture via hand-written classes or std::bind

## Item 33: Use decltype on auto&& parameters to std::forward them

### generic lambdas

- at least **C++14**
- use `auto` in their parameter specifications
- **operator()** in the lambda's closure class is a **template**

### Snippets

- C++14 lambdas can be variadic:

  ```c++
  auto f = [](auto&&... params)
  {
      return func(std::forward<decltype(params)>(params)...);
  };
  ```

### Key points

- Use **decltype** on auto&& parameters to std::forward them

## Item 34: Prefer lambdas to std::bind

### Function objects

- ref: [Function objects - cppreference.com](https://en.cppreference.com/w/cpp/utility/functional)

### Constrained situations

- in **C++11**

- **Move capture**. C++11 lambdas don’t offer move capture, but it can be emulated through **a combination of a lambda and std::bind**

- **Polymorphic function objects**. Because the function **call operator on a bind object uses perfect forwarding**, it can accept arguments of any type:

  ```c++
  class PolyWidget 
  {
  public:
      template<typename T>
      void operator()(const T& param);
  };
  
  PolyWidget pw;
  auto boundPW = std::bind(pw, _1);
  ```

### Snippets

Given the following function to set up an alarm:

```c++
using Time = std::chrono::steady_clock::time_point;
enum class Sound { Beep, Siren, Whistle };
using Duration = std::chrono::steady_clock::duration;
enum class Volume { Normal, Loud, LoudPlusPlus };

void setAlarm(Time t, Sound s, Duration d);
void setAlarm(Time t, Sound s, Duration d, Volume v);
```

- one lambda example in **c++14**:

  ```c++
  /** L for lambda */
  auto setSoundL = [](Sound s)
  {
      using namespace std::chrono;
      using namespace std::literals; // for C++14 suffixes
      
      setAlarm(steady_clock::now() + 1h, s, 30s);
  };
  ```

- the corresponding bind example in **c++14**:

  ```c++
  using namespace std::chrono;
  using namespace std::literals;
  using namespace std::placeholders; // needed for use of "_1"
  
  using SetAlarm3ParamType = void(*)(Time t, Sound s, Duration d);
  
  /** B for bind */
  auto setSoundB = std::bind(
      static_cast<SetAlarm3ParamType>(setAlarm), // otherwise bind will not know which one should take
      std::bind(std::plus<>(), steady_clock::now(), 1h), // use a defer eval for the timing
      _1,
      30s
  );
  ```

- **std::bind always copies its arguments**, but callers can achieve the effect of having an argument stored by reference by applying `std::ref` to it:

  ```c++
  auto compressRateB = std::bind(compress, std::ref(w), _1);
  ```

### Key points

- Lambdas are more readable, more expressive, and may be more efficient than using `std::bind`
- **In C++11 only**, std::bind may be useful for implementing **move capture** or for binding objects with **templatized function call operators**
