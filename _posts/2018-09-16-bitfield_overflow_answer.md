---
layout: default
categories: [poll answers] 
title: C++ Poll Answer for September 16th 2018 
---

This is implementation defined behavior, [\[expr.post.incr\]p1](http://eel.is/c++draft/expr.post.incr#1.sentence-11) says

>The value of a postfix ++ expression is the value of its operand. [ Note: The value obtained is a copy of the original value
— end note] The operand shall be a modifiable lvalue.
The type of the operand shall be an arithmetic type other than cv bool, or a pointer to a complete object type.
The value of the operand object is modified by adding 1 to it.
The value computation of the ++ expression is sequenced before the modification of the operand object.
With respect to an indeterminately-sequenced function call, the operation of postfix ++ is a single evaluation.
[ Note: Therefore, a function call shall not intervene between the lvalue-to-rvalue conversion and the side effect associated with any single postfix ++ operator.
— end note]
The result is a prvalue. The type of the result is the cv-unqualified version of the type of the operand.
**If the operand is a bit-field that cannot represent the incremented value, the resulting value of the bit-field is implementation-defined.**
See also [expr.add] and [expr.ass]. 

See it [live on Wandbox with UBSan](https://wandbox.org/permlink/C64BalMhwI7NYRyG)

Also see [See DR 1816: Unclear specification of bit-field values](http://www.open-std.org/jtc1/sc22/wg21/docs/cwg_defects.html#1816). 
