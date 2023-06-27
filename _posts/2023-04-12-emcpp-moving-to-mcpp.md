---
title: Moving to Modern C++
author: ipa
date: 2023-04-12
categories: [C-Cpp Language]
tags: [coding, modern cpp, cpp]
---

## Item 7: Distinguish between () and {} when creating objects

### Key points

- Braced initialization is the most widely usable initialization syntax, it prevents narrowing conversions, and it’s immune to C++’s most vexing parse.
- During constructor overload resolution, braced initializers are matched to `std::initializer_list` parameters if at all possible, even if other constructors offer seemingly better matches.
- An example of where the choice between parentheses and braces can make a significant difference is creating a `std::vector<numeric type>` with two arguments
- Choosing between parentheses and braces for object creation inside templates can be challenging.

## Item 8: Prefer nullptr to 0 and NULL

### Snippets

- mutex guard function execution:

  ```c++
  using MuxGuard = std::lock_guard<std::mutex>;
  
  template<typename FuncType, typename MuxType, typename PtrType>
  decltype(auto) lockAndCall(FuncType func, MuxType& mutex, PtrType ptr)
  {
      MuxGuard g(mutex);
      return func(ptr);
  }
  
  bool f(Widget* pw);
  std::mutex fm;
  auto result3 = lockAndCall(f, fm, nullptr);
  ```

### Key points

- Prefer `nullptr` to `0` and `NULL`.  
- Avoid overloading on integral and pointer types  

## Item 9: Prefer alias declarations to typedefs

### Snippets

- alias templates:

  ```c++
  template<typename T>
  using MyAllocList = std::list<T, MyAlloc<T>>;
  
  /** use case */
  template<typename T>
  class Widget 
  {
  private:
      MyAllocList<T> list;
  };
  ```

- typedef templates (a workaround)

  ```c++
  template<typename T>
  struct MyAllocList 
  {
      typedef std::list<T, MyAlloc<T>> type;
  };
  
  /** use case */
  template<typename T>
  class Widget 
  {
  private:
      typename MyAllocList<T>::type list;    // don't forget the typename and ::type
  };
  ```

- convert from typedef templates to alias templates:

  ```c++
  template<typename T>
  using MyAllocList_t = typename MyAllocList<T>::type;
  ```

### Key points

- typedefs don’t support templatization, but alias declarations do
- Alias templates avoid the **::type** suffix and, in templates, the **typename** prefix often required to refer to typedefs.
- **C++14** offers alias templates for all the **C++11** type traits transformations  

## Item 10: Prefer scoped enums to unscoped enums

### Snippets

- convert to underlying type of enum at compile time:

  ```c++
  template<typename E>
  constexpr auto toUType(E enumerator) noexcept
  {
      return static_cast<std::underlying_type_t<E>>(enumerator);
  }
  ```
- use enum to get element from a tuple-ish data structure:

  ```c++
  using UserInfo = std::tuple<std::string, std::string, std::size_t>;
  
  UserInfo uInfo;
  namespace UserInfoFields
  {
      enum : std::size_t { uiName, uiEmail, uiReputation };
  }
  /** instead of std::get<1>(uInfo) */
  auto val = std::get<toUType(UserInfoFields::uiEmail)>(uInfo);
  ```


### Key points

- C++98-style enums are now known as **unscoped** enums.
- Enumerators of scoped enums are visible only within **the enum**. They convert to other types only with a cast.
- Both scoped and unscoped enums support specification of the **underlying type**. The default underlying type for scoped enums is `int`. Unscoped enums have no default underlying type.
- Scoped enums may always be forward-declared. **Unscoped enums may be forward-declared only if their declaration specifies an underlying type**

## Item 11: Prefer deleted functions to private undefined ones

### Snippets

- delete some instantiations for a template function:

  ```c++
  class Widget 
  {
  public:
      template<typename T>
      void processPointer(T* ptr);
  }
  
  template<>
  void Widget::processPointer<void>(void*) = delete;
  template<>
  void Widget::processPointer<const void>(const void*) = delete;
  ```

  this cannot done by partially declare them as private, because it’s not possible to give a member function template specialization a different access level from that of the main template.   

### Key points

- Prefer deleted functions to private undefined ones
- Any function may be deleted, including non-member functions and template instantiations

## Item 12: Declare overriding functions override

### Overriding requirements

- The base class function must be virtual

- The base and derived function names must be identical (except in the case of destructors)

- The parameter types of the base and derived functions must be identical

- The const-ness of the base and derived functions must be identical

- The return types and exception specifications of the base and derived functions must be compatible

- (new in C++11)The functions' **reference qualifiers** must be identical

  ```c++
  class Widget 
  {
  public:
      void doWork() &; // this version of doWork applies only when *this is an lvalue
      void doWork() &&; // this version of doWork applies only when *this is an rvalue
  };
  ```

### final keyword

- Applying final to a virtual function prevents the function from being overridden in derived classes. 
- final may also be applied to a class, in which case the class is prohibited from being used as a base class.

### Key points

- Declare overriding functions `override` 
- Member function reference qualifiers make it possible to treat lvalue and rvalue objects (\***this**) differently

## Item 13: Prefer const_iterators to iterators

### Snippets

- insert a value to a vector if a specific spot is found, otherwise append:

  ```c++
  /** C++14 version of findAndInsert */
  template<typename C, typename V>
  void findAndInsert(C& container, const V& targetVal, const V& insertVal)
  {
      using std::cbegin;
      using std::cend;
      
      auto it = std::find(cbegin(container), cend(container), targetVal);
      container.insert(it, insertVal);
  }
  ```

- Add const iterator in C++11:

  ```c++
  template <class C>
  auto cbegin(const C& container)->decltype(std::begin(container))
  {
      return std::begin(container);
  }
  ```

### Key points

- Prefer **const_iterator**s to **iterator**s  
- In maximally generic code, prefer non-member versions of `begin`, `end`, `rbegin`, etc., over their member function counterparts

## Item 14: Declare functions noexcept if they won't emit exceptions

### Snippets

- different style of noexcept function declaration:

  ```c++
  int f(int x) throw(); // no exceptions from f: C++98 style
  int f(int x) noexcept; // no exceptions from f: C++11 style
  ```

  If, at runtime, an exception leaves **f**, **f**'s exception specification is violated:

  - With the **C++98** exception specification, the call stack is unwound to **f**'s caller, and after some actions not relevant here, program execution is terminated
  - With the **C++11** exception specification, runtime behavior is slightly different: the stack is ***only possibly*** unwound before program execution is terminated

- conditional noexcept:

  ```c++
  template <class T, size_t N>
  void swap(T (&a)[N], T (&b)[N]) noexcept(noexcept(swap(*a, *b)));
  
  template<class T1, class T2>
  struct pair 
  {
      void swap(pair& p) noexcept(noexcept(swap(first, p.first)) && noexcept(swap(second, p.second)));
  };
  ```

- explicitly express a dtor can emit exception:

  ```c++
  class C
  {
  public:
      virtual ~C() noexcept(false);
  }
  ```

### Key points

- `noexcept` is part of a function's interface, and that means that callers may depend on it
- `noexcept` functions are more optimizable than non-noexcept functions
- `noexcept` is particularly valuable for the **move operations, swap, memory deallocation functions, and destructors**
- Most functions are exception-neutral rather than `noexcept`  

## Item 15: Use constexpr whenever possible

### constexpr function

- `constexpr` functions can be used in contexts that demand compile-time constants. If the values of the arguments you pass to a `constexpr` function in such a context are known during compilation, the result will be computed during compilation. If any of the arguments' values is not known during compilation, the code will be rejected
- When a `constexpr` function is called with one or more values that are not known during compilation, it acts like a normal function, computing its result at runtime. This means no need two functions to perform the same operation, one for compile-time constants and one for all other values. The `constexpr`function does it all

### Snippets

- constexpr pow function:

  ```c++
  constexpr int pow(int base, int exp) noexcept
  {
      return (exp == 0 ? 1 : base * pow(base, exp - 1));
  }
  ```

- a data struct for 2D point:

  ```c++
  class Point 
  {
  public:
      constexpr Point(double xVal = 0, double yVal = 0) noexcept : x(xVal), y(yVal) {}
      
      constexpr double xValue() const noexcept { return x; }
      constexpr double yValue() const noexcept { return y; }
      
      constexpr void setX(double newX) noexcept { x = newX; } // constexpr for void allowed in c++14
      constexpr void setY(double newY) noexcept { y = newY; } // constexpr for void allowed in c++14
  private:
      double x, y;
  };
  ```

### Key points

- `constexpr` objects are const and are initialized with values known during compilation
- `constexpr` functions can produce compile-time results when called with arguments whose values are known during compilation
- `constexpr` objects and functions may be used in a wider range of contexts than non-constexpr objects and functions
- `constexpr` is part of an object's or function's interface

## Item 16: Make const member functions thread safe

### Data race

- different threads reading and writing the same memory without synchronization, leads to undefined behavior

### Key points

- Make `const` member functions thread safe unless it's certain they'll never be used in a concurrent context
- Use of `std::atomic` variables may offer better performance than a mutex, but they’re suited for manipulation of only a single variable or memory location

## Item 17: Understand special member function generation

### Special member functions

The generated special member functions are implicitly public and inline, and they're **nonvirtual unless the function in question is a destructor** in a derived class inheriting from a base class with a virtual destructor  

- **C++98**: the default constructor, the destructor, the copy constructor, the copy assignment operator
  - **Rule of Three**: if you declare any of a copy constructor, copy assignment operator, or destructor, you should declare all three  
- **C++11**: the default constructor, the destructor, the copy constructor, the copy assignment operator, **the move constructor**, **the move assignment operator**
  - **move** is a request, **copy** is a fallback
  - declare one **move** operation will stop compiler auto-generating the other, and will stop compiler auto-generate copy operations
  - declare one **copy** operation will **NOT** stop compiler auto-generating the other, and will stop compiler auto-generate move operations
  - to auto-generate move operations:
    - No **copy** operations are declared in the class
    - No **move** operations are declared in the class
    - No **destructor** is declared in the class  

### Snippets

- special member function signatures:

  ```c++
  class Base 
  {
  public:
      virtual ~Base() = default; // make dtor virtual
      
      Base(Base&&) = default; // support moving
      Base& operator=(Base&&) = default;
      
      Base(const Base&) = default; // support copying
      Base& operator=(const Base&) = default;
  };
  ```

### Key points

- The special member functions are those compilers may generate on their own: default constructor, destructor, copy operations, and move operations
- Move operations are generated only for classes lacking explicitly declared move operations, copy operations, and a destructor
- The copy constructor is generated only for classes lacking an explicitly declared copy constructor, and it's deleted if a move operation is declared. The copy assignment operator is generated only for classes lacking an explicitly declared copy assignment operator, and it's deleted if a move operation is declared. Generation of the copy operations in classes with an explicitly declared destructor is deprecated
- Member function templates never suppress generation of special member functions
