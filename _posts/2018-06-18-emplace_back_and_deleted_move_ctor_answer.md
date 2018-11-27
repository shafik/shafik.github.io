---
layout: default
title: C++ Poll Answer for June 18th 2018 
---

The program invokes undefined behavior per [\[sequence.reqmts\] Table 82](http://eel.is/c++draft/sequence.reqmts#tab:containers.sequence.optional) which contains the following for `emplace_back`:

>Requires:  T shall be Cpp17EmplaceConstructible into X from args.  
>**For vector, T shall also be Cpp17MoveInsertable into X**.
>Returns: a.back().

and [\[res.on.required\]p1](http://eel.is/c++draft/res.on.required#1):

>Violation of any preconditions specified in a function's Requires: element results in undefined behavior unless the function's Throws: element specifies throwing an exception when the precondition is violated.

Emplacing does not need it but in order to resize the vector, the type need to support move or copy.

What is fascinating here, is that violating a requires paragraph is not ill-formed which requires a diagnostic but undefined behavior. I think this explanation from std-discussion mailing list topic [Violation of "Requires:" paragraph results in undefined behavior?](https://groups.google.com/a/isocpp.org/forum/#!msg/std-discussion/n7UHcUQtcQk/Oque11CjBgAJ):

>Even basic requirements like MoveAssignable has both a syntactic and a semantic component.

>The syntactic component is that `a=rv` must be well-formed, where rv is an rvalue expression of the type.
The semantic component is that it must do what is advertised - i.e., after the expression is evaluated, a must be equivalent to the value of rv before the expression is evaluated.

>You can't diagnose violation of the semantic requirements, at least without heroic efforts. And while you can conceivably split the arguably diagnosable syntactic components out, it is a giant amount of effort for very little actual gain, especially with Concepts on the horizon. 
