---
layout: default
categories: [poll answers] 
title: C++ Poll Answer for February 6th 2019 
---

They are different in C and C++ and valid. In C++ it will be:

    int foo(int&&)

in C it will be

    int foo(int)

Obligatory godbolts, [For C](godbolt.org/z/7090d4):

    int foo( int and );
    int (*funptr)(int) = foo;

and [for C++](godbolt.org/z/Crrsrf).

    int foo( int and );
    int (*funptr)(int&&) = foo;

This is due to how alternative tokens(spellings) are handled in C++ and C. In C++ we get them out of the box [\[lex.digraph\]p2](http://eel.is/c++draft/lex.digraph#2):

>In all respects of the language, each alternative token behaves the same, respectively, as its primary token, except for its spelling.12
The set of alternative tokens is defined in Table 1.

and **Table 1** includes:

>     Alternative    Primary
>       and            &&
	

but in C we need to include **iso646.h** see [7.9 Alternative spellings \<iso646.h\>](https://port70.net/~nsz/c/c11/n1570.html#7.9):

> The header <iso646.h> defines the following eleven macros (on the left) that expand to the corresponding tokens (on the right):
>
>     and        &&
    and_eq     &=
    bitand     &
    bitor      |
    compl      ~
    not        !
    not_eq     !=
    or         ||
    or_eq      |=
    xor        ^
    xor_eq     ^=

We can see including that header:

    #include <iso646.h>

    int foo( int and ){}
    int (*funptr)(int) = foo;

[breaks the example](https://godbolt.org/z/RsUxUt)
