---
title: Moving to Modern C++
author: ipa
date: 2023-04-12
categories: [C-Cpp Language]
tags: [coding, modern cpp, cpp]
---

## Distinguish between () and {} when creating objects

### Key points

- Braced initialization is the most widely usable initialization syntax, it prevents narrowing conversions, and it’s immune to C++’s most vexing parse.
- During constructor overload resolution, braced initializers are matched to `std::initializer_list` parameters if at all possible, even if other constructors offer seemingly better matches.
- An example of where the choice between parentheses and braces can make a significant difference is creating a `std::vector<numeric type>` with two arguments
- Choosing between parentheses and braces for object creation inside templates can be challenging.

## Prefer nullptr to 0 and NULL

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

## Prefer alias declarations to typedefs

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

## Prefer scoped enums to unscoped enums

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

## Prefer deleted functions to private undefined ones

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
