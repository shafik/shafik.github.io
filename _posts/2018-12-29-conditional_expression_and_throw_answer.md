---
layout: default
categories: [poll answers] 
title: C++ Poll Answer for December 29th 2018 
---

Prior to [defect report 1550](http://www.open-std.org/jtc1/sc22/wg21/docs/cwg_defects.html#1550) and [defect report 1560](http://www.open-std.org/jtc1/sc22/wg21/docs/cwg_defects.html#1560) the answer would have been `1` because if the second or third operand of a *conditional-expression* was a *throw-expression* the result would be *prvalue* which would mean a temporary would be created. DR 1560 refers to this as gratuitous and surprising:

>A glvalue appearing as one operand of a conditional-expression in which the other operand is a throw-expression is converted to a prvalue, regardless of how the conditional-expression is used:
>
>> If either the second or the third operand has type void, then the lvalue-to-rvalue (7.1 [conv.lval]), array-to-pointer (7.2 [conv.array]), and function-to-pointer (7.3 [conv.func]) standard conversions are performed on the second and third operands, and one of the following shall hold:
>>
>>- The second or the third operand (but not both) is a throw-expression (18.1 [except.throw]); the result is of the type of the other and is a prvalue.
>
>This seems to be gratuitous and surprising.

This potential incompatibility is also covered in [\[diff.cpp11.expr\]](https://timsong-cpp.github.io/cppwp/n4659/diff.cpp11.expr):

>**Change**: A conditional expression with a throw expression as its second or third operand keeps the type and value category of the other operand.   
>**Rationale**: Formerly mandated conversions (lvalue-to-rvalue, array-to-pointer, and function-to-pointer standard conversions), especially the creation of the temporary due to lvalue-to-rvalue conversion, were considered gratuitous and surprising.   
>**Effect on original feature**: Valid C++ 2011 code that relies on the conversions may behave differently in this International Standard:  
>
>     struct S {
       int x = 1;
       void mf() { x = 2; }
     };
     int f(bool cond) {
       S s;
       (cond ? s : throw 0).mf();
       return s.x;
     }
>
>In C++ 2011, f(true) returns 1. In this International Standard, it returns 2.  
>
>     sizeof(true ? "" : throw 0)
>
>In C++ 2011, the expression yields sizeof(const char\*). In this International Standard, it yields sizeof(const char[1]).


clang applies this all the way back to C++98 mode [see live godbolt](https://godbolt.org/z/T3VSWQ) but it looks like gcc [has a bug](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=64372) and this currently does not compile on trunk.
