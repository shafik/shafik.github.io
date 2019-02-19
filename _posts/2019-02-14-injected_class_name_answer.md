---
layout: default
categories: [poll answers] 
title: C++ Poll Answer for February 14th 2019 
---

B is invalid see  [\[class.qual\]p2](http://eel.is/c++draft/basic.lookup#class.qual-2):

>In a lookup in which function names are not ignored28 and the nested-name-specifier nominates a class C: 
>
>- if the name specified after the nested-name-specifier, when looked up in C, is the injected-class-name of C ([class]), or
>
>- in a using-declarator of a using-declaration ([namespace.udecl]) that is a member-declaration, if the name specified after the nested-name-specifier is the same as the identifier or the simple-template-id's template-name in the last component of the nested-name-specifier,
>
>the name is instead considered to name the constructor of class C. [ Note: For example, the constructor is not an acceptable
> lookup result in an elaborated-type-specifier so the constructor would not be used in place of the injected-class-name.— end note]
>Such a constructor name shall be used only in the declarator-id of a declaration that names a constructor or in a using-declaration.
>[ Example:
>
>      struct A { A(); };
>      struct B: public A { B(); };
>
>      A::A() { }
>      B::B() { }
>
>      B::A ba;            // object of type A
>      A::A a;             // error, A​::​A is not a type name
>      struct A::A a2;     // object of type A
>
> — end example]


The class name is injected into the scope of the class itself but depending on the context the nested-name-specifier refers
to the constructor not the type

Also see [Defect Report 147](http://open-std.org/jtc1/sc22/wg21/docs/cwg_defects.html#147)

