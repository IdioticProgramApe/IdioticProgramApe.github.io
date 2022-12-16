---
layout: default
title: Type Traits
parent: C++
grand_parent: C/C++
nav_order: 1
permalink: /docs/c-c++/c++/type-traits/
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Overview

A basic trait for types is the categories in which they can be classified. This is a chart on how these categories overlap:

<table>
<tr><th></th><th>primary categories</th><th colspan="3">composite categories</th></tr>
<tr><td rowspan="4">fundamental</td><td>void</td><td></td><td></td><td></td></tr>
<tr><td><tt>std::nullptr_t</tt></td><td></td><td rowspan="7">scalar</td><td rowspan="10">object</td></tr>
<tr><td>integral</td><td rowspan="2">arithmetic</td></tr>
<tr><td>floating point</td></tr>
<tr><td rowspan="10">compound</td><td>pointer</td><td></td></tr>
<tr><td>member object pointer</td><td rowspan="2">member pointer</td></tr>
<tr><td>member function pointer</td></tr>
<tr><td>enum</td><td></td></tr>
<tr><td>union</td><td></td><td></td></tr>
<tr><td>class<span title="(a class that is not a union)">*</span></td><td></td><td></td></tr>
<tr><td>array</td><td></td><td></td></tr>
<tr><td>l-value reference</td><td rowspan="2">reference</td><td></td><td></td></tr>
<tr><td>r-value reference</td><td></td><td></td></tr>
<tr><td>function</td><td></td><td></td><td></td></tr>
</table>

\* = excluding unions

## Helper Classes

Standard classes to assist in creating compile-time constants.

| <!-- -->                                                     | <!-- -->                           |
| ------------------------------------------------------------ | ---------------------------------- |
| [**integral_constant**](https://cplusplus.com/reference/type_traits/integral_constant/) | Integral constant (class template) |
| [**true_type**](https://cplusplus.com/reference/type_traits/true_type/) | True type (class)                  |
| [**false_type**](https://cplusplus.com/reference/type_traits/false_type/) | False type (class)                 |

## Type Traits

Classes to obtain characteristics of types in the form of compile-time constant values.

### Primary Type Categories

| <!-- --> | <!-- -->|
| ----- | ------ |
|[**is_array**](https://cplusplus.com/reference/type_traits/is_array/)	|Is array (class template)|
|[**is_class**](https://cplusplus.com/reference/type_traits/is_class/)	|Is non-union class (class template)|
|[**is_enum**](https://cplusplus.com/reference/type_traits/is_enum/)	|Is enum (class template)|
|[**is_floating_point**](https://cplusplus.com/reference/type_traits/is_floating_point/)	|Is floating point (class template)|
|[**is_function**](https://cplusplus.com/reference/type_traits/is_function/)	|Is function (class template)|
|[**is_integral**](https://cplusplus.com/reference/type_traits/is_integral/)	|Is integral (class template)|
|[**is_lvalue_reference**](https://cplusplus.com/reference/type_traits/is_lvalue_reference/)	|Is lvalue reference (class template)|
|[**is_member_function_pointer**](https://cplusplus.com/reference/type_traits/is_member_function_pointer/)	|Is member function pointer (class template)|
|[**is_member_object_pointer**](https://cplusplus.com/reference/type_traits/is_member_object_pointer/)	|Is member object pointer (class template)|
|[**is_pointer**](https://cplusplus.com/reference/type_traits/is_pointer/)	|Is pointer (class template)|
|[**is_rvalue_reference**](https://cplusplus.com/reference/type_traits/is_rvalue_reference/)	|Is rvalue reference (class template)|
|[**is_union**](https://cplusplus.com/reference/type_traits/is_union/)	|Is union (class template)|
|[**is_void**](https://cplusplus.com/reference/type_traits/is_void/)	|Is void (class template)|

### Composite Type Categories

| <!-- --> | <!-- -->|
| ----- | ------ |
|[**is_arithmetic**](https://cplusplus.com/reference/type_traits/is_arithmetic/)	|Is arithmetic type (class template)|
|[**is_compound**](https://cplusplus.com/reference/type_traits/is_compound/)	|Is compound type (class template)|
|[**is_fundamental**](https://cplusplus.com/reference/type_traits/is_fundamental/)	|Is fundamental type (class template)|
|[**is_member_pointer**](https://cplusplus.com/reference/type_traits/is_member_pointer/)	|Is member pointer type (class template)|
|[**is_object**](https://cplusplus.com/reference/type_traits/is_object/)	|Is object type (class template)|
|[**is_reference**](https://cplusplus.com/reference/type_traits/is_reference/)	|Is reference type (class template)|
|[**is_scalar**](https://cplusplus.com/reference/type_traits/is_scalar/)	|Is scalar type (class template)|

### Type Properties

| <!-- --> | <!-- -->|
| ----- | ------ |
|[**is_abstract**](https://cplusplus.com/reference/type_traits/is_abstract/)	|Is abstract class (class template)|
|[**is_const**](https://cplusplus.com/reference/type_traits/is_const/)	|Is const-qualified (class template)|
|[**is_empty**](https://cplusplus.com/reference/type_traits/is_empty/)	|Is empty class (class template)|
|[**is_literal_type**](https://cplusplus.com/reference/type_traits/is_literal_type/)	|Is literal type (class template)|
|[**is_pod**](https://cplusplus.com/reference/type_traits/is_pod/)	|Is POD type (class template)|
|[**is_polymorphic**](https://cplusplus.com/reference/type_traits/is_polymorphic/)	|Is polymorphic (class template)|
|[**is_signed**](https://cplusplus.com/reference/type_traits/is_signed/)	|Is signed type (class template)|
|[**is_standard_layout**](https://cplusplus.com/reference/type_traits/is_standard_layout/)	|Is standard-layout type (class template)|
|[**is_trivial**](https://cplusplus.com/reference/type_traits/is_trivial/)	|Is trivial type (class template)|
|[**is_trivially_copyable**](https://cplusplus.com/reference/type_traits/is_trivially_copyable/)	|Is trivially copyable (class template)|
|[**is_unsigned**](https://cplusplus.com/reference/type_traits/is_unsigned/)	|Is unsigned type (class template)|
|[**is_volatile**](https://cplusplus.com/reference/type_traits/is_volatile/)	|Is volatile-qualified (class template)|

### Type Features

| <!-- --> | <!-- -->|
| ----- | ------ |
|[**has_virtual_destructor**](https://cplusplus.com/reference/type_traits/has_virtual_destructor/)	|Has virtual destructor (class template)|
|[**is_assignable**](https://cplusplus.com/reference/type_traits/is_assignable/)	|Is assignable (class template)|
|[**is_constructible**](https://cplusplus.com/reference/type_traits/is_constructible/)	|Is constructible (class template)|
|[**is_copy_assignable**](https://cplusplus.com/reference/type_traits/is_copy_assignable/)	|Is copy assignable (class template)|
|[**is_copy_constructible**](https://cplusplus.com/reference/type_traits/is_copy_constructible/)	|Is copy constructible (class template)|
|[**is_destructible**](https://cplusplus.com/reference/type_traits/is_destructible/)	|Is destructible (class template)|
|[**is_default_constructible**](https://cplusplus.com/reference/type_traits/is_default_constructible/)	|Is default constructible (class template)|
|[**is_move_assignable**](https://cplusplus.com/reference/type_traits/is_move_assignable/)	|Is move assignable (class template)|
|[**is_move_constructible**](https://cplusplus.com/reference/type_traits/is_move_constructible/)	|Is move constructible (class template)|
|[**is_trivially_assignable**](https://cplusplus.com/reference/type_traits/is_trivially_assignable/)	|Is trivially assignable (class template)|
|[**is_trivially_constructible**](https://cplusplus.com/reference/type_traits/is_trivially_constructible/)	|Is trivially constructible (class template)|
|[**is_trivially_copy_assignable**](https://cplusplus.com/reference/type_traits/is_trivially_copy_assignable/)	|Is trivially copy assignable (class template)|
|[**is_trivially_copy_constructible**](https://cplusplus.com/reference/type_traits/is_trivially_copy_constructible/)	|Is trivially copy constructible (class template)|
|[**is_trivially_destructible**](https://cplusplus.com/reference/type_traits/is_trivially_destructible/)	|Is trivially destructible (class template)|
|[**is_trivially_default_constructible**](https://cplusplus.com/reference/type_traits/is_trivially_default_constructible/)	|Is trivially default constructible (class template)|
|[**is_trivially_move_assignable**](https://cplusplus.com/reference/type_traits/is_trivially_move_assignable/)	|Is trivially move assignable (class template)|
|[**is_trivially_move_constructible**](https://cplusplus.com/reference/type_traits/is_trivially_move_constructible/)	|Is trivially move constructible (class template)|
|[**is_nothrow_assignable**](https://cplusplus.com/reference/type_traits/is_nothrow_assignable/)	|Is assignable throwing no exceptions (class template)|
|[**is_nothrow_constructible**](https://cplusplus.com/reference/type_traits/is_nothrow_constructible/)	|Is constructible throwing no exceptions (class template)|
|[**is_nothrow_copy_assignable**](https://cplusplus.com/reference/type_traits/is_nothrow_copy_assignable/)	|Is copy assignable throwing no exceptions (class template)|
|[**is_nothrow_copy_constructible**](https://cplusplus.com/reference/type_traits/is_nothrow_copy_constructible/)	|Is copy constructible throwing no exceptions (class template)|
|[**is_nothrow_destructible**](https://cplusplus.com/reference/type_traits/is_nothrow_destructible/)	|Is nothrow destructible (class template)|
|[**is_nothrow_default_constructible**](https://cplusplus.com/reference/type_traits/is_nothrow_default_constructible/)	|Is default constructible throwing no exceptions (class template)|
|[**is_nothrow_move_assignable**](https://cplusplus.com/reference/type_traits/is_nothrow_move_assignable/)	|Is move assignable throwing no exception (class template)|
|[**is_nothrow_move_constructible**](https://cplusplus.com/reference/type_traits/is_nothrow_move_constructible/)	|Is move constructible throwing no exceptions (class template)|

### Type Relationships

| <!-- --> | <!-- -->|
| ----- | ------ |
|[**is_base_of**](https://cplusplus.com/reference/type_traits/is_base_of/)	|Is base class of (class template)|
|[**is_convertible**](https://cplusplus.com/reference/type_traits/is_convertible/)	|Is convertible (class template)|
|[**is_same**](https://cplusplus.com/reference/type_traits/is_same/)	|Is same type (class template)|

### Property Queries

| <!-- --> | <!-- -->|
| ----- | ------ |
|[**alignment_of**](https://cplusplus.com/reference/type_traits/alignment_of/)	|Alignment of (class template)|
|[**extent**](https://cplusplus.com/reference/type_traits/extent/)	|Array dimension extent (class template)|
|[**rank**](https://cplusplus.com/reference/type_traits/rank/)	|Array rank (class template)|

## Type Transformations

Classes to obtain new types by applying specific transformations to existing types.

### Const-Volatile Qualifications

| <!-- --> | <!-- -->|
| ----- | ------ |
|[**add_const**](https://cplusplus.com/reference/type_traits/add_const/)	|Add const qualification (class template)|
|[**add_cv**](https://cplusplus.com/reference/type_traits/add_cv/)	|Add const volatile qualification (class template)|
|[**add_volatile**](https://cplusplus.com/reference/type_traits/add_volatile/)	|Add volatile qualification (class template)|
|[**remove_const**](https://cplusplus.com/reference/type_traits/remove_const/)	|Remove const qualification (class template)|
|[**remove_cv**](https://cplusplus.com/reference/type_traits/remove_cv/)	|Remove cv qualification (class template)|
|[**remove_volatile**](https://cplusplus.com/reference/type_traits/remove_volatile/)	|Remove volatile qualification (class template)|

### Compound Type Atterations

| <!-- --> | <!-- -->|
| ----- | ------ |
|[**add_pointer**](https://cplusplus.com/reference/type_traits/add_pointer/)	|Add pointer (class template)|
|[**add_lvalue_reference**](https://cplusplus.com/reference/type_traits/add_lvalue_reference/)	|Add lvalue reference (class template)|
|[**add_rvalue_reference**](https://cplusplus.com/reference/type_traits/add_rvalue_reference/)	|Add rvalue reference (class template)|
|[**decay**](https://cplusplus.com/reference/type_traits/decay/)	|Decay type (class template)|
|[**make_signed**](https://cplusplus.com/reference/type_traits/make_signed/)	|Make signed (class template)|
|[**make_unsigned**](https://cplusplus.com/reference/type_traits/make_unsigned/)	|Make unsigned (class template)|
|[**remove_all_extents**](https://cplusplus.com/reference/type_traits/remove_all_extents/)	|Remove all array extents (class template)|
|[**remove_extent**](https://cplusplus.com/reference/type_traits/remove_extent/)	|Remove array extent (class template)|
|[**remove_pointer**](https://cplusplus.com/reference/type_traits/remove_pointer/)	|Remove pointer (class template)|
|[**remove_reference**](https://cplusplus.com/reference/type_traits/remove_reference/)	|Remove reference (class template)|
|[**underlying_type**](https://cplusplus.com/reference/type_traits/underlying_type/)	|Underlying type of enum (class template)|

### Other Type Generators
| <!-- --> | <!-- -->|
| ----- | ------ |
|[**aligned_storage**](https://cplusplus.com/reference/type_traits/aligned_storage/)	|Aligned storage (class template)|
|[**aligned_union**](https://cplusplus.com/reference/type_traits/aligned_union/)	|Aligned union (class template)|
|[**common_type**](https://cplusplus.com/reference/type_traits/common_type/)	|Common type (class template)|
|[**[**conditional**](https://cplusplus.com/reference/type_traits/conditional/)**](https://cplusplus.com/reference/type_traits/common_type/)	|Conditional type (class template)|
|[**enable_if**](https://cplusplus.com/reference/type_traits/enable_if/)	|Enable type if condition is met (class template)|
|[**result_of**](https://cplusplus.com/reference/type_traits/result_of/)	|Result of call (class template)|

## Notes

This page is a copy of [type traits](https://cplusplus.com/reference/type_traits/) for the sake of future search, for more details please go to the original site.