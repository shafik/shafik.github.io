---
layout: default
title: C++ Poll Answer for November 6th 2018 
twitter_share_title: CPlusPlus Poll Answer for November 6th 2018 
---

It is ill-formed, reinterpret_cast is not allowed to cast away constness [\[expr.reinterpret.cast\]p2](http://eel.is/c++draft/expr.reinterpret.cast#2)

>The reinterpret\_­cast operator shall not cast away constness.
>An expression of integral, enumeration, pointer, or pointer-to-member type can be explicitly converted to its own type; such a cast yields the value of its operand.

Using it over C-style cast catches errors and clarifies intent `-Wold-style-cast` FTW!

[Obligatory godbolt](godbolt.org/z/RbUQ5g)
