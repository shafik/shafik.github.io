---
layout: default
categories: [poll answers] 
title: C++ Poll Answer for April 12th 2019 
---

The answer to this is C, it is conditionally supported. A compiler could make this ill-formed or well-defined.

Looking at [\[lex.ccon\]p2](http://eel.is/c++draft/lex.ccon#2):

>... An ordinary character literal that **contains more than one c-char is a multicharacter literal**.
A multicharacter literal, or an ordinary character literal containing a single c-char not representable in the execution character set, is conditionally-supported, has type int, and **has an implementation-defined value**. 

tells us multicharacter literals have implementation defined values. But we also have to look at [\[lex.ccon\]p7](http://eel.is/c++draft/lex.ccon#7):

>...Escape sequences in which the character following the backslash is not listed in Table 8 are conditionally-supported, with implementation-defined semantics...

which tells us that escape sequences not mentioned in the table are conditionally supported.

So while

```
\77
```

Is a valid octal escape sequence and has a value of `63`

```
\97
```

Is not a valid escape sequence it looks like clang and gcc just ignore the `\` and treat it like `'97'` and then we end up with a implementation defined multicharacter literal value.
It looks like [clang](https://godbolt.org/z/cmNl4P) generates a value of `14647` which does not fit in a *char* and is truncated to a value of
`55` So we end up with `55 < 63` Which in this case is *true* but is not portable.

The [gcc manual describes](https://gcc.gnu.org/onlinedocs/gcc-4.7.4/cpp/Implementation-defined-behavior.html) how gcc calculates the value of multicharacter literal.
