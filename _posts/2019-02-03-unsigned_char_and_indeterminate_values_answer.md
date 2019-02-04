---
layout: default
categories: [poll answers] 
title: C++ Poll Answer for February 3rd 2019 
---

The value of `d` is indeterminate.

You may expect it to be undefined behavior but the standard provides for several exceptions for when using an indeterminate value is not undefined behavior, see [\[basic.indet\]p2](http://eel.is/c++draft/basic.indet#2):

>if an indeterminate value is produced by an evaluation, the behavior is undefined except in the following cases: 
>
> ...
>
>- If an indeterminate value of unsigned ordinary character type or std::byte type is produced by the evaluation of the right operand of a simple assignment operator ([expr.ass]) whose first operand is an lvalue of unsigned ordinary character type or std::byte type, an indeterminate value replaces the value of the object referred to by the left operand.
>
> ...
>
> [Example:
>
>     int f(bool b) {
       unsigned char c;
       unsigned char d = c;          // OK, d has an indeterminate value
       int e = d;                    // undefined behavior
       return b ? d : 0;             // undefined behavior if b is true
>     }
> 
>— end example]

This was clarified because of [defect report 1787](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2014/n3833.html#1787) also see [GB comment 2](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2014/n3903.html#GB2) And my [Stackoverflow answer](https://stackoverflow.com/a/23415662/1708801) which also covers some C background.
