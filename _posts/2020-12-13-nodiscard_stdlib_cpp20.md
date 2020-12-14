---
layout: post
categories: [C++]
title: C++20 and [[nodiscard]] in the library
---

[In quiz #113](https://twitter.com/shafikyaghmour/status/1335053054828191744) we examined the following code:


```
#include <vector>

int main() {
  std::vector<int>{}.empty(); // Should we recieve a diagnostic here?
                              // Diagnsotic is either a warning or an error.
}
```

In [the answer](https://twitter.com/shafikyaghmour/status/1336874248548220929) I noted that in C++20 we added [\[\[nodiscard\]\]](https://en.cppreference.com/w/cpp/language/attributes/nodiscard) to many functions in the standard library including `std::vector::empty`. If we use `[[nodiscard]]` when declaring a function, it means that if we ignore the return value of the function (with some exceptions) we may receive a diagnostic (either a warning or an error). 

`[[nodiscard]]` is an interesting feature because [the standard does not require a diagnostic](http://eel.is/c++draft/dcl.attr.nodiscard#4) if the caller ignores the return value of a function marked `[[nodiscard]]`. It merely strongly encourages it. If we look at the proposal for the feature [\[\[unused\]\], \[\[nodiscard\]\] and
\[\[fallthrough\]\] attributes](http://wg21.link/p0068r0) the rationale for this was:

>the purpose of warnings is to
>catch mistakes, but there are many times when an implementation cannot definitively determine whether a
>piece of code is intentional or accidental.


Now if we check the main compilers out there we do indeed see that they choose to [issue a diagnostic for this case](https://godbolt.org/z/WE9Grn).

I also noted that this was somewhat of a trick question because implementations could choose to issue a diagnostic before C++20 as well. This all falls under the rubric of Quality of Implementation (QoI). The standard does not talk about [warning or errors, merely diagnostics](http://eel.is/c++draft/intro.defs#defns.diagnostic) and the implementations make choices as to the trade-offs and whether to make certain constructs that require a diagnostic a hard error or a warning. 

The standard also allows the implementation to have their own implementation defined attributes e.g.:

```
[[clang::optnone]] void f(); // See https://clang.llvm.org/docs/AttributeReference.html#optnone
```

The standard says that the implementation [should ignore attributes they don't recognize](http://eel.is/c++draft/dcl.attr#grammar-6).

Talking about QoI, compilers often try to warn us about potentially problematic code even though they may strictly be conforming according to the standard. One example is a `switch` statement where the cases do not cover the whole range of enum values:

```
enum E {A,B,C};

void f() {
  E e;  
  switch (e) {
  case A:
    //...
    break;
  case B:
    //...
    break;
  // We have not handles the E::C case
  }
}
```

[live example](https://godbolt.org/z/33eKn8). This is well-formed according the standard but likely and error that we are not covering one of the enumerators in the switch statement. This will also catch the case where we extend the enum to include more enumerators but forget to update all the switches. So many compilers choose to warn and should give us the ability to turn off the warning if we so choose.
