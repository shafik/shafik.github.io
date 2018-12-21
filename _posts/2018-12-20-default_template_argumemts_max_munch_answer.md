---
layout: default
categories: [poll answers] 
title: C++ Poll Answer for December 12th 2018 
---

This is not well formed code, this is another variation on the maximal munch problem which is also present in this [Stack Overflow question](https://stackoverflow.com/q/28354108/1708801).

We can see this is not well-formed from [\[temp.param\]p15](http://eel.is/c++draft/temp.param#15) which says:

>When parsing a default template-argument for a non-type template-parameter, the first non-nested > is taken as the end of the template-parameter-list rather than a greater-than operator.
>[ Example:
>
>      template<int i = 3 > 4 >        // syntax error
      class X { /* ... */ };
>
>      template<int i = (3 > 4) >      // OK
      class Y { /* ... */ };
>
>— end example]

[Obligatory godbolt](https://godbolt.org/z/h2UrcP)
