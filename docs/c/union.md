---
layout: default
title: Union
parent: C
nav_order: 2
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

|                           | STRUCTURE                                                    | UNION                                                        |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Keyword                   | `struct`                                                     | `union`                                                      |
| Size                      | the size of structure is **greater than or equal to** the sum of sizes of its members | the size of union is **equal to** the size of its largest member |
| Memory                    | each member within a structure is assigned unique storage area of location | memory allocated is shared by individual members of union    |
| Value Altering            | altering the value of a member will not affect other members of the structure | altering the value of any of the member will alter other member values |
| Accessing members         | individual member can be accessed at a time                  | only one member can be accessed at a time                    |
| Initialization of Members | several members of a structure can initialize at once        | only the **first member** of a union can be initialized      |

- A union is a derived type (similar to a structure) with members that share the same storage space
  - sometimes the same type of construct needs different types of data
- Used mainly in advanced programming applications where it is necessary to store different types of data in the same storage area
  - can be used to save space and for alternating data
  - a union does not waste storage on variables that are not being used
- Each element in a union is called member
  - can define a union with many members
    - only one member can contain a value at any given time, so only one access of a member at a given time
  - the members of a union can be of any data type
    - in most cases, unions contain two or more data types
- It is programmer's responsibility to ensure that the data in the union is referenced with the proper data type
  - referencing data in a union with a variable of the wrong type is a **logic error**
- The operations that can be performed on a union are:
  - assignment: assign a union to another union of the same type
  - &: take the address of a union variable
  - access: access union members

## How to use

### Defining a Union

- the union definition is normally placed in a header and included in all source files that use the union type
- when a union is defined, it creates a user-defined type
  - no memory is allocated
  - to allocate memory for a given union type and work with it, we need to create variables
- a union can be defined to contain as many members as desired
- structures can be defined that contain unions, as can arrays
- pointers to unions can also be declared

### Accessing Union Members

- same as struct, but members are mutually exclusive, once another member got assigned, previous member is no longer valid
- be careful with the member type when accessing it

## Example

```C
#include <stdio.h>

union car 
{
    int i_value;
    float f_value;
    char c_value[40];
} g_car1, g_car2, *g_car3; // these cars are global

struct owner
{
    char social_security[12];
};

struct leasecompany
{
    char name[40];
    char headquarters[40];
};

struct car_data
{
    char make[15];  // padding to 16
    int status;

    // c11 introduced anonymous union
    union // no name after union, cannot be used outside of car_data struct
    {
        struct owner owncar;
        struct leasecompany leasecar;
    } data;
} cars[40];

union mixed
{
    int i;
    char c;
    float f;
};

int main()
{
    // local declaration
    union car car1, car2, *car3; 

    // %zu is for size_t (introduced <= C89) which introduced in C99
    printf("Memory size occupied by data: %zu\n", sizeof(union car));   // 40
    printf("Memory size occupied by data: %zu\n", sizeof(cars));        // 4000 = 100 * 40
	
    // constructor, use first member type
    union mixed x = {.i = 15};
    printf("Int = %d\n", x.i);
    printf("Character = %c\n", x.c);   // invalid value
    printf("Float = %f\n", x.f);       // invalid value

    x.c = 'j';
    printf("Int = %d\n", x.i);         // invalid value
    printf("Character = %c\n", x.c);
    printf("Float = %f\n", x.f);       // invalid value
    
    x.f = 3.14f;
    printf("Int = %d\n", x.i);         // invalid value
    printf("Character = %c\n", x.c);   // invalid value
    printf("Float = %f\n", x.f);
	
    // copy constructor
    union mixed x_copy = x;
    printf("Int = %d\n", x.i);         // invalid value
    printf("Character = %c\n", x.c);   // invalid value
    printf("Float = %f\n", x.f);
    
    return 0;
}
```
