---
layout: default
title: C++ Poll Answer for June 26th 2018 
---

Unclear! Although evidence is it was meant to be well defined, we could justify a few diff positions, see Stack Overflow question [Is it defined behavior to reference an early member from a later member expression during aggregate initialization?](https://stackoverflow.com/q/32940847/1708801) for some more details.

For C++20 [P0329 Designated Initialization](https://wg21.link/p0329) if we look at [\[dcl.init.aggr\]p6](http://eel.is/c++draft/dcl.init.aggr#6) it now says:

>The initializations of the elements of the aggregate are evaluated in the element order.
That is, all value computations and side effects associated with a given element are sequenced before those of any element that follows it in order.

Why can I use foo.i in the declaration? mystruct?

    foo{45, foo.i};
    ————————^

This is covered in [\[basic.scope.pdecl\]p1](http://eel.is/c++draft/basic.scope.pdecl#1) says:

>The point of declaration for a name is immediately after its complete declarator and before its initializer (if any), except as noted below.
[ Example:

    unsigned char x = 12;
    { unsigned char x = x; }

>Here the second x is initialized with its own (indeterminate) value.— end example]

Which is the source for some interesting questions, for example:

- [Does initialization entail lvalue-to-rvalue conversion? Is `int x = x;` UB?](https://stackoverflow.com/q/14935722/1708801)
- [Has C++ standard changed with respect to the use of indeterminate values and undefined behavior in C++14?](https://stackoverflow.com/q/23415661/1708801)
