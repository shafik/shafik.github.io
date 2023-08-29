---
layout: post
categories: [C++]
title: Auto Auto Auto
---

# Auto Auto Auto
In my [C++ quiz #237](https://hachyderm.io/@shafik/110929184328245345) I asked the following question

```cpp
#237 Given the following in C++:

struct A{};
using T = auto() -> auto(*)() -> auto(*)() -> A;

// Without checking, is this well-formed?
```

The short answer is, Yes. The likely follow-up question is, what does `T` represent?

## Peeling the Onion

Often when we have a complicated declaration, it is helpful to break it down into smaller pieces and build up to
the final declaration. Kind of like peeling back the layers of an onion or in this case adding a layer one at a time.
In that vain we can start with the following declaration for `T`:

```cpp
struct A{};
using T = auto() -> A;
```

Although, it might look a bit more familiar if we compare it to this code:

```cpp
struct A{};

// Using trailing return type syntax
// A function f
// returning a type A
auto f() -> A;
```

Here `f` is a function returning an `A` type. The *auto* here just means we can use the trailing return type syntax to denote
the return type which in this case is `A`. Below we have a function `g` which can take as an argument a pointer to a function returning an `A`. It is worth noting that functions decay to pointer to functions when passed by value. So when we pass a function as a argument it will be as a pointer to a function:

```cpp
// A function returning an A
A f(){return {};}

// A function taking as an argument
// a function pointer to a function
// returnung a type A
void g(T);

// An example of calling g with f
void h() {
  g(f);
}
```

## Layer 2

Now we are going to create our pattern and add another trailing return type layer, which gives us type `T3`. We use type `T2` just to demonstrate a stepping stone between `T` and `T3`:

```cpp
struct A{};
using T = auto() -> A;

// A pointer to a function
// returning type A
using T2 = auto(*)() -> A;

// A function
// returning a function pointer to a function
// returning a type A
using T3 = auto() -> auto(*)() -> A;
```

and here is how we could use it. We have `f` which is a function that returns a type `A`. We then have `f3` which is a function that returns a function pointer to a function that returns an `A` and `g3` which is a function that can take an `f3` as an argument:

```cpp
A f(){return {};}

// A function
// returning a pointer to a function
// returning type A
T2 f3() { return f; }

void g(T);

// A function taking as an argument
// A function returning a function pointer
// to a function returning a type A
void g3(T3);

void h() {
  g(f);
  g3(f3);
}
```

## Layer 3

Now we can add the final layer which gets us to the original code the question was about. All we are doing is adding another funcition pointer layer. We will again use a stepping stone type, in this case `T4`:

```cpp
// We have seen this previously
struct A{};
using T = auto() -> A;
using T2 = auto(*)() -> A;
using T3 = auto() -> auto(*)() -> A;

// A pointer to a function
// returning a function pointer to a function
// returning a type A
using T4 = auto(*)() -> auto(*)() -> A;

// A function
// returning a pointer to a function
// returning a pointer to a function
// returning a type A
using T5 = auto() -> auto(*)() -> auto(*)() -> A;
```

Which we could use as follows:

```cpp
A f(){return {};}
T2 f3() { return f; }
T4 f5() { return f3; }


void g(T);
void g3(T3);
void g5(T5);


void h() {
  g(f);
  g3(f3);
  g5(f5);
}
```

h/t [Cameron DaCamara](https://twitter.com/starfreakclone) who [posted this tweet](https://twitter.com/starfreakclone/status/1655713819451338752) which puzzled me greatly until I broke it down for myself
and figured it out and then turned it into a quiz to puzzle everyone else.
