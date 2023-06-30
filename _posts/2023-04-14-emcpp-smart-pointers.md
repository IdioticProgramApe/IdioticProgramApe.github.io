---
title: Smart Pointers
author: ipa
date: 2023-04-14
categories: [C-Cpp Language]
tags: [coding, modern cpp, cpp]
mermaid: true
img_path: /moderncpp/
---

## Item 18: Use std::unique_ptr for exclusive-ownership resource management

### Snippets

- Given a structure as below (Investment has a virtual dtor):

  ```mermaid
  stateDiagram-v2
  	classI: Investment
  	classS: Stock
  	classB: Bond
  	classR: RealEstate
  	
  	classS --> classI : Inheritance
  	classB --> classI : Inheritance
  	classR --> classI : Inheritance
  ```

  ```c++
  // c++14, return type deduction auto
  template<typename... Ts>
  auto makeInvestment(Ts&&... params)
  {
      auto delInvmt = [](Investment* pInvestment)
      {
          makeLogEntry(pInvestment);
      };
      
      std::unique_ptr<Investment, decltype(delInvmt)> pInv(nullptr, delInvmt);
      if ( ... )
      {
          pInv.reset(new Stock(std::forward<Ts>(params)...));
      }
      else if ( ... )
      {
          pInv.reset(new Bond(std::forward<Ts>(params)...));
      }
      else if ( ... )
      {
          pInv.reset(new RealEstate(std::forward<Ts>(params)...));
      }
      return pInv;
  }
  ```

### Key points

- `std::unique_ptr` is a small, fast, **move-only** smart pointer for managing resources with exclusive-ownership semantics
- By default, resource destruction takes place via delete, but custom deleters can be specified. Stateful deleters and function pointers as deleters increase the size of `std::unique_ptr` objects
- Converting a `std::unique_ptr` to a `std::shared_ptr` is easy

## Item 19: Use std::shared_ptr for shared-ownership resource management  

### Reference count

- **std::shared_ptrs are twice the size of a raw pointer**, because they internally contain a raw pointer to the resource as well as a raw pointer to the resource's reference count
- **Memory for the reference count must be dynamically allocated**. Conceptually, the reference count is associated with the object being pointed to, but pointed-to objects know nothing about this. They thus have no place to store a reference count. (A pleasant implication is that any object even those of built-in types may be managed by std::shared_ptrs.) The cost of the dynamic allocation is avoided when the std::shared_ptr is created by std::make_shared, but there are situations where std::make_shared can't be used. Either way, the reference count is stored as dynamically allocated data
- **Increments and decrements of the reference count must be atomic**, because there can be simultaneous readers and writers in different threads. For example, a std::shared_ptr pointing to a resource in one thread could be executing its destructor (hence decrementing the reference count for the resource it points to), while, in a different thread, a std::shared_ptr to the same object could be copied (and therefore incrementing the same reference count). Atomic operations are typically slower than non-atomic operations, so even though reference counts are usually only a word in size, you should assume that reading and writing them is comparatively costly

### Memory Management

- custom deleter:

  ```c++
   // custom deleter
  auto loggingDel = [](Widget *pw)
  {
      makeLogEntry(pw);
      delete pw;
  };
  
  std::unique_ptr<Widget, decltype(loggingDel)> upw(new Widget, loggingDel);
  std::shared_ptr<Widget> spw(new Widget, loggingDel);	// deleter not a part of the ptr type
  ```

- pointer structure:

  ![shared_ptr_memory_management](shared_ptr_memory_management.png){: width="413" height="262" }

  _shared pointer memory management_

### Control block

- **std::make_shared always creates a control block**. It manufactures a new object to point to, so there is certainly no control block for that object at the time `std::make_shared` is called
- **A control block is created when a std::shared_ptr is constructed from a unique-ownership pointer** (i.e., a `std::unique_ptr` or `std::auto_ptr`). Unique-ownership pointers don't use control blocks, so there should be no control block for the pointed-to object. (As part of its construction, the `std::shared_ptr` assumes ownership of the pointed-to object, so the uniqueownership pointer is set to null.)
- **When a std::shared_ptr constructor is called with a raw pointer, it creates a control block**. If you wanted to create a `std::shared_ptr` from an object that already had a control block, you'd presumably pass a `std::shared_ptr` or a `std::weak_ptr` as a constructor argument, not a raw pointer. `std::shared_ptr` constructors taking std::shared_ptrs or std::weak_ptrs as constructor arguments don't create new control blocks, because they can rely on the smart pointers passed to them to point to any necessary control blocks

### Snippets

- shared ptr to factory class:

  ```c++
  class Widget: public std::enable_shared_from_this<Widget>
  {
  public:
      // factory function that perfect-forwards args to a private ctor
      template<typename... Ts>
      static std::shared_ptr<Widget> create(Ts&&... params);
  
      void process()
      {
          ...    // process the Widget
          
          // add std::shared_ptr to current object to processedWidgets
          processedWidgets.emplace_back(shared_from_this());
      }
      
  private:
      ...  // ctors
  };
  ```

### Key points

- std::shared_ptrs offer convenience approaching that of **garbage collection** for the shared lifetime management of arbitrary resources
- Compared to `std::unique_ptr`, `std::shared_ptr` objects are typically twice as big, incur overhead for control blocks, and require atomic reference count manipulations
- Default resource destruction is via delete, but custom deleters are supported. **The type of the deleter has no effect on the type of the std::shared_ptr**
- Avoid creating std::shared_ptrs from variables of raw pointer type

## Item 20: Use std::weak_ptr for std::shared_ptr like pointers that can dangle

### Snippets

- check weak_ptr whether it's valid or get the shared_ptr out of it:

  ```c++
  auto spw = std::make_shared<Widget>();
  std::weak_ptr<Widget> wpw(spw);
  spw = nullptr;    // now the weak pointer dangles
  
  if(wpw.expired())
  {
      try
      {
          // if weak_ptr expired, throw a bad weak ptr exception
          std::shared_ptr<Widget> spw3(wpw);
      }
      except(std::bad_weak_ptr& e)
      {
          ... // handle the exception
      }
  }
  else
  {
      std::shared_ptr<Widget> spw1 = wpw.lock();    // get the shared_ptr to dereference the obj
      if(spw1 != nullptr)
      {
          ...   // process the Widget
      }
  }
  ```

- a quick-and-dirty implementation of a **caching factory** method:

  ```c++
  std::shared_ptr<const Widget> fastLoadWidget(WidgetID id)
  {
      static std::unordered_map<WidgetID, std::weak_ptr<const Widget>> cache;
      
      // objPtr is std::shared_ptr to cached object (or null if object's not in cache)
      auto objPtr = cache[id].lock();
      
      // if not in cache, load and cache it, otherwise return it
      if (!objPtr) 
      {
          objPtr = loadWidget(id);
          cache[id] = objPtr;
      }
      return objPtr;
  }
  ```

### Key points

- Use std::weak_ptr for std::shared_ptr-like pointers that can dangle
- Potential use cases for std::weak_ptr include caching, observer lists, and the prevention of std::shared_ptr cycles

## Item 21: Prefer std::make_unique and std::make_shared to direct use of new

### Snippets

- make_unique in C++11 (introduced in C++14), ref,[make_unique (Revision 1)](https://isocpp.org/files/papers/N3656.txt):

  ```c++
  template<typename T, typename... Ts>
  std::unique_ptr<T> make_unique(Ts&&... params)
  {
      return std::unique_ptr<T>(new T(std::forward<Ts>(params)...));
  }
  ```

- use braced initializers to create a shared_ptr, braced initializers can't be perfect-forwarded:

  ```c++
  // create std::initializer_list
  auto initList = { 10, 20 };
  // create std::vector using std::initializer_list ctor
  auto spv = std::make_shared<std::vector<int>>(initList);
  ```

### Key points

- Compared to direct use of new, make functions eliminate source code duplication, improve exception safety, and, for `std::make_shared` and `std::allocate_shared`, generate code that's smaller and faster
- Situations where use of make functions is inappropriate include the need to specify **custom deleters** and a desire to pass **braced initializers**
- For `std::shared_ptrs`, additional situations where make functions may be ill-advised include:
  - classes with **custom memory management**
  - systems with memory concerns, very **large objects**, and std::weak_ptrs that outlive the corresponding std::shared_ptrs

## Item 22: When using the Pimpl Idiom, define special member functions in the implementation file

### Key points

- The Pimpl Idiom decreases build times by reducing compilation dependencies between class clients and class implementations
- For `std::unique_ptr` pImpl pointers, declare special member functions in the class header, but implement them in the implementation file. Do this even if the default function implementations are acceptable
- The above advice applies to `std::unique_ptr`, but not to `std::shared_ptr`

