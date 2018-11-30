---
layout: default
categories: [poll answers] 
title: C++ Poll Answer for June 15th 2018 
---

The [C++20 Designated Initialization proposal](https://wg21.link/p0329) says that A and B are valid, not C and D.

The rationale is covered in the proposal and says which added [\[diff.dcl\]p10](http://eel.is/c++draft/diff.dcl#10):

>**Affected subclause**: [dcl.init.aggr]  
>**Change**: In C++, designated initialization support is restricted compared to the corresponding functionality in C.
In C++, designators for non-static data members must be specified in declaration order, designators for array elements and nested designators are not supported, and designated and non-designated initializers cannot be mixed in the same initializer list.
> Example: 
>
    struct A { int x, y; };
    struct B { struct A a; };
    struct A a = {.y = 1, .x = 2};  // valid C, invalid C++
    int arr[3] = {[1] = 5};         // valid C, invalid C++
    struct B b = {.a.x = 0};        // valid C, invalid C++
    struct A c = {.x = 1, 2};       // valid C, invalid C++
>
>**Rationale**: In C++, members are destroyed in reverse construction order and the elements of an initializer list are evaluated in lexical order, so field initializers must be specified in order.
Array designators conflict with lambda-expression syntax.
Nested designators are seldom used.  
>**Effect on original feature**: Deletion of feature that is incompatible with C++.  
>**Difficulty of converting**: Syntactic transformation.  
>**How widely used**: Out-of-order initializers are common.
The other features are seldom used.

