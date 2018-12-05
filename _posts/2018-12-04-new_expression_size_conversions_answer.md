---
layout: default
categories: [poll answers] 
title: C++ Poll Answer for December 4th 2018 
---

Only A is well formed, [the grammar](http://eel.is/c++draft/expr.new#nt:noptr-new-declarator):

    noptr-new-declarator:
      [ expression ] attribute-specifier-seq<sub>opt</sub>
	  noptr-new-declarator [ constant-expression ] attribute-specifier-seq<sub>opt</sub>

tells us the first size is an *expression* while subsequent sizes are *constant expression*.

From [\[expr.new\]p7](http://eel.is/c++draft/expr.new#7) we see the expression has to be implicitly convertible to size\_t
While the constant expression shall be a converted constant expression of type std::size\_t

>Every constant-expression in a noptr-new-declarator shall be a converted constant expression of type std::size\_t and
shall evaluate to a strictly positive value. The expression in a noptr-new-declarator is implicitly converted to std::size\_t.
\[Example: Given the definition int n = 42, new float[n][5] is well-formed (because n is the expression of a noptr-new-declarator), 
but new float[5][n] is ill-formed (because n is not a constant expression).
— end example\] 

which does not allow [floating point to floating point to integral conversions](http://eel.is/c++draft/conv.fpint#1):

godbolt [Case A](https://godbolt.org/z/jbP3Z3) and [Case B](https://godbolt.org/z/pM7LRU)
