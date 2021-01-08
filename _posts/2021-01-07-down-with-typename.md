---
layout: post
categories: [C++]
title: Down with typename
---

In [C++ quiz quiz #117](https://twitter.com/shafikyaghmour/status/1345087081438068736) I asked in C++20 where was `typename` required in code below:

```
template<class T> struct S {
  typename/*A*/ T::R f(typename/*B*/ T::P p) { 
    typename/*C*/ T::x *i;
    return {};
  } 
};
```

and the answer is that it is required in position `C`. 

Previous to C++20 it would have been required in all three places but the proposal [Down with typename!](http://wg21.link/p0634)
changed that. Previously as noted in the proposal:

>If X<T>::Y — where T is a template parameter — is to denote a type, it must be preceded by the keyword typename; otherwise, it is assumed to denote a name producing an expression. There are currently two notable exceptions to this rule: base-specifiers and mem-initializer-ids. For example:
>
>
>  ```
>  template<class T>
>    struct D: T::B { // No typename required here. 
>  };
>  ```
> Clearly, no typename is needed for this base-specifier because nothing but a type is possible in that context. 
  
 The proposal also notes that there are several places where only a *type* is possible and seeks to remove the requirement to 
 use `typename` in all such locations. This change for example allows us to avoid using typename in many locations in the following code ([see it live](https://godbolt.org/z/o1sxTo)):
 
```
template<class T> T::R f(); // OK, return type of a function declaration at global scope
template<class T> struct S {
  using Ptr = PtrTraits<T>::Ptr; // OK, in a defining-type-id
  T::R f(T::P p) { // OK, class scope
    return static_cast<T::R>(p); // OK, type-id of a static_cast
  }
  auto g() -> S<T*>::Ptr;  // OK, trailing-return-type
};

template<typename T> void f() {
  void (*pf)(T::X); // Variable pf of type void* initialized with T::X
}
```

So why can't we avoid using `typename` in all cases? For C++20 [\[temp.res\]p6](https://timsong-cpp.github.io/cppwp/n4861/temp.res#6) says:

>A qualified-id that refers to a member of an unknown specialization, that is not prefixed by typename, and that is not otherwise assumed to name a type (see above) denotes a non-type.

There may be contexts in which it is ambiguous if we are referring to a *type* or an *expression*, for example ([see it live](https://godbolt.org/z/EsPv5n)):

```
template <class T> void f(int i) {
  T::x * i;  // This will be assumed to be the expression
             //   T::x multiplied by i
             // Not a declaration of variable i of 
             //   type pointer to T::x?
}

struct Foo {
 typedef int x;
};

struct Bar {
  static int const x = 5;
};

int main() {
  f<Bar>(1);          // OK
  f<Foo>(1);          // error: Foo::x is a type
}
```

So in those contexts it will be assumed that it is not a *type*. Other cases can be seen below ([see it live](https://godbolt.org/z/raz3s9)):

```
#include <iostream>
#include <vector>
 
int p = 1;
template <typename T>
void foo(const std::vector<T> &v)
{
    // std::vector<T>::const_iterator is a dependent name
    typename std::vector<T>::const_iterator it = v.begin(); // typename required
 
    typename std::vector<T>::const_iterator typedef iter_t; // typename required
}
