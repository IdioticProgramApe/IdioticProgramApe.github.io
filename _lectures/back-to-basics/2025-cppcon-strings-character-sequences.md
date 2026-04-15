# C++ Strings and Character Sequences

Ref: [Back To Basics: C++ Strings and Character Sequences - Nicolai Josuttis - CppCon 2025](https://www.youtube.com/watch?v=LVU7lq9aL8o)

[TOC]

## Introduction

### Literals

with the auto statement: `auto c = <Example>;`

| Example | Type            | Notes                                                        |
| ------- | --------------- | ------------------------------------------------------------ |
| `'h'`   | `char`          | - **read-only** character literal<br />- integral value of type `char` |
| `"hi"`  | `const char[3]` | - **read-only** string literal<br />- type is `const char[3]` with fixed array of constant chars, including `\0` at the end<br />- decayed as `const char*` (as auto would do), pointing to the first character |

### std::string

- a class, which represents dynamic sequence of characters
- can be modified (values and length)
- can be empty (initial state)

### std::string_view

- introduced in **C++17** as `const char*` with `std::string` interface
- a class, which only provide read-only API, doesn't own the data
- data pointer pointing to the first character of the string slice, also a size descriptor to defined how long the slice is
- note string view:
  - might not have `\0` at the end
  - might have no value (`data() == nullptr`), unlike literal string will always have a value `\0`

## Raw String Literals

| Example     | Output | Notes                          |
| ----------- | ------ | ------------------------------ |
| `"\"\\n\""` | "\n"   | Using `\` to translate         |
| `R"("\n")"` | "\n"   | Using Raw string (since C++11) |

Format of raw string: `R"delim(...)delim"`

- `delim` is totally optional, can be empty and can have up to 16 basic characters except parenthesis, backslash, whitespace
- `)delim"` is used to terminate the raw string

## Strings

Referring `std::string` by default in this section.

There are 2 parts in a `std::string` object, one is the raw string data (***<u>memory allocated on heap</u>***), the other one is a descriptor, aka a struct contains 3 information, the **size** of the string data, the **capacity** of the allocated memory for the string data and a **data pointer** which point to the first character of the string data.

- null terminator `\n` doesn’t count in string size.

- avoid string copying, since memory allocation will occur again.
- reassign a longer string value can trigger both memory deallocation and allocation

### SSO

a.k.a **Small/Short String Optimization**, the data pointer is now become a fixed sized char array: `char[16]` (up to compiler)

```c++
void func()
{
    // sso
    std::string s;
    std::cout << s.size() << ", " << s.capacity() << std::endl;
}
```

The output would be **0** and **15**.

- if we reassign a string which larger than 16 bytes, a memory allocation happens, and the data now stores the pointer pointing to the new string instead of the small string itself.
- if we re-reassign a small string (less than 15 bytes), the data will not directly store it, instead this small string will be written into the previously allocated memory.
  - but if we copy this string object to another one, we found that the small string is still in the data variable.

### std::string vs. std::string_view

```c++
void foo(const std::string& s);
void bar(std::string_view sv);

foo("hello, world");    // compute length, potential memory allocation (sso), copy chars
bar("hello, world");    // compute length only (pointing to some text in .rodata segment)

std::string s {"hello, world"};
bar(s);                 // implicit conversion from std::string to std::string_view (pointing to the heap allocated text)
```

- When use string view from a string, make sure the string exists and without memory reallocation.
- we prefer the string view if:
  - local read access only
  - no null terminator `\0` required (no c function call)
  - lifetime is ok
  - `data()` checks for `nullptr`

> `std::string_view` can be regarded as raw pointer! Use it carefully, all best practices with raw pointer also applied to it:
>
> - don’t return `std::string_view` object, unless you checked for the lifetime.
>
> - a bad example:
>
>   ```c++
>   std::string operator+(std::string_view sv1, std::string_view sv2)
>   {
>       return std::string(sv1) + std::string(sv2);
>   }
>     
>   template<typename T>
>   T sum(const T& x, const T& y)
>   {
>       return x + y;
>   }
>     
>   std::string_view sv = "hi";
>   auto xy = sum(sv, sv);  // 💣
>   ```

## Internationalization (I18N)

Fun fact: 18 here stands for the amount of letter between the letter `i` and the letter `n`, there is another one called `L10N`, which stands for `localization`.

`UTF-8`: use different amount of bytes to represent different character, based on their rarity. This encoding indeed can encode all charsets however, it’s not very friendly if we want to find the i-th character in one string, because the size is not fixed.

- it’s good for storing and exchanging data
- it’s **NOT** good for parsing data

| Charset    | C++ Standard | Prefix       | Notes                                                        |
| ---------- | ------------ | ------------ | ------------------------------------------------------------ |
| `char`     | All versions |              |                                                              |
| `wchar_t`  | All versions | `L`          | - not define whether 2 or 4 bytes (up to compiler, `-fshort-wchar`)<br />- primarily used on windows |
| `char8_t`  | C++ 20       | `u8` (C++17) | - before C++20, use `char`<br />- `char8_t` is one byte of a character, not one character |
| `char16_t` | C++ 11       | `u`          |                                                              |
| `char32_t` | C++ 11       | `U`          |                                                              |

> String type are template:
>
> ```
> std::string ->
> std::basic_string<char> ->
> std::basic_string<char, std::char_traits<char>> ->
> std::basic_string<char, std::char_traits<char>, std::allocator<char>>
> ```
>
> we have different string types
>
> | String Type      | String Template               |
> | ---------------- | ----------------------------- |
> | `std::string`    | `std::basic_string<char>`     |
> | `std::wstring`   | `std::basic_string<wchar_t>`  |
> | `std::u16string` | `std::basic_string<char16_t>` |
> | `std::u32string` | `std::basic_string<char32_t>` |
>
> this is same for the string view.