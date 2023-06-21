---
title: auto
author: ipa
date: 2023-04-11
categories: [C-Cpp Language]
tags: [coding, modern cpp, cpp]
---

## Prefer auto to explicit type declarations

### Snippets

- replace a long complex type name:

  ```c++
  /** 
   * algorithm to dwim ("do what I mean") for all elements in range from b to e
   */
  template<typename It> 
  void dwim(It b, It e)
  {
      while (b != e) 
      {
          // use auto instead of: typename std::iterator_traits<It>::value_type
          auto currValue = *b;
      }
  }
  ```

- replace a unsure type name:

  ```c++
  std::unordered_map<std::string, int> m;
  
  // use auto instead of: std::pair<const std::string, int>, it's const!
  for (const auto& p : m) {}
  ```

### Key points

- `auto` variables must be initialized, are generally immune to type mismatches that can lead to portability or efficiency problems, can ease the process of refactoring, and typically require less typing than variables with explicitly specified types.
- auto-typed variables are subject to the pitfalls described in Items 2 and 6.

## Use the explicitly typed initializer idiom when auto deduces undesired types

### Snippets

- c++ forbids references to bits:

  ```c++
  std::vector<bool> features(5, false);
  
  // use explicit type cast with auto to guide the type deduction
  auto feature = static_cast<bool>(features[0]);
  ```

### Key points

- "Invisible" proxy types can cause `auto` to deduce the "wrong" type for an initializing expression.
- The explicitly typed initializer idiom forces auto to deduce the type you want it to have. using `static_cast`

