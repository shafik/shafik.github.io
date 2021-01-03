---
layout: post
categories: [C++]
title: Is Zero a Butterfly?
---

Is Zero a Butterfly? Wait? What? That sounds like a pretty odd question to ask for C++ but let's explore a little and 
see where it takes us. 

So what is zero in C++? Perhaps the most obvious answer would be it is an *integer* and while this is true it misses a bigger
story about zeros super powers in C++.

What is super about zero?

Well the first some fun trivia. Zero is an *octal literal*,
see the [grammar for integer literals](http://eel.is/c++draft/lex.icon#nt:octal-literal):

```
octal-literal:
  0
  octal-literal 'opt octal-digit
```

It is only single digit *octal literal*, it won't help you do anything special but might impress some of your friends.

Zero is a kind of super-literal (not official terminology), it can be converted to all sort of types, some mundane but many 
quite surprising. Most would not be surprised that the following code is valid:

```
void f(int);
f(0); // Zero is an integer, not surprising.
f(1); // Nope, not surprising either.
```

Zero is an *integer* after all and we will probably not be surprised the following works either [(see it live)](https://godbolt.org/z/rd1a6d):

```
void f1(bool);
void f2(char);
f1(0); // Ok, we are used to non-zero values being true and zero values being false.
f2(0); // Perhaps a little icky but char is an integer type.
f2(1); // Why not 1 as well?
```

We are probably used to thinking of zero values as converting to `false` and non-zero values converting to `true`,
so not likely surprising. We might not be as thrilled about zero converting to a *char* without any explicit conversions
but *char*, *signed char* and *unsigned char* are all [integer types](http://eel.is/c++draft/basic.types#basic.fundamental-1).

So how about *pointers*? Is zero a *pointer*? How about non-zero values? Are non-zero *integers* also *pointers*? Are we 
surprised by the following [(see it live)](https://godbolt.org/z/G56vso)?:

```
f(int *p);
f(0); // Ok, zero is a null pointer constant.
f(1); // Error, 1 is not a valid pointer constant.
f(reinterpret_cast<int*>(1)); // Ok (although questionable) implementation-defined 
                              // mapping from integer to pointer.
```

So now we start seeing some interesting behavior (*powers*), zero is special, [\[conv.ptr\]p1](http://eel.is/c++draft/conv.ptr#1) tells us that:

> A null pointer constant is an integer literal ([lex.icon]) with *value zero* or a prvalue of type std::nullptr_t. 

but other *integers* do not have this super ability and if we want to convert them to *pointers* we must use 
`reinterpret_cast`. This will be a questionable but implementation defined mapping. We inherit this directly from C and
pre-C++11 we [used to allow constant expressions that evaluated to zero to be also be consider null pointer constants](https://twitter.com/shafikyaghmour/status/1156425516057952256?s=20).
So the following prior to C++11 was also valid [(see it live)](https://godbolt.org/z/sjMWaW):

```
f(int *p);
f( 1 - 1 );  // Ok, prior to C++11, 1-1 is an integral constant expression.           
f( ~-1 );    // Also ok prior to C++11!
```

How about arrays? Is zero an array (😱) [(see it live)](https://godbolt.org/z/Wv7463)? 

```
void f(char arr[5]){} // Identical to
                      // void f(char *arr);
f(0); // Ok, zero is a null pointer constant
```

Ok, this probably feels like cheating but it works and is likely pretty surprising. Here we are seeing the effect of
a parameter of type *array of T* being adjusted to *pointer to T*, see [\[dcl.fct\]p5](http://eel.is/c++draft/dcl.fct#5):

> After determining the type of each parameter, any parameter of type “array of T” or of function type T is adjusted to be “pointer to T”.

This is another thing that C++ has inherited directly from C. In C++ we can avoid this adjustment by passing array by reference. This is used for example by [std::end to determine the size of the array](https://stackoverflow.com/a/33496357/1708801):

```
template< class T, std::size_t N >
T* end( T (&array)[N] );
```

but C-syle arrays are not idiomatic C++ and we should prefer using `std::array` or `std::vector` but often for generic 
code we need to support C-style arrays, as the `std::end` example above.

Ok, so what about `std::string`? You may be saying, no way, zero can't be a `std::string` can it? 
Let's see [(see it live)](https://godbolt.org/z/KbzWf6):

```
void f(std::string s){}
f(0);             // Ok, zero converts to a const char *, may crash due to undefined behavior
f("hello world"); // Ok, a string literal will decay to a pointer to a const char 
f(reinterpret_cast<const char *>(1)); // Undefined behavior, 1 is not a valid pointer to a const char 
```

What we are seeing here is that [std::string](https://en.cppreference.com/w/cpp/string/basic_string/basic_string) has a constuctor that takes a *const char\** (I am being lose here, really should be *CharT*) and copies the contents:

```
constexpr basic_string( const CharT* s,
                        const Allocator& alloc = Allocator() );
```

Due to the legacy C conversion of zero to a *null pointer constant* we can convert zero to a *const char\** and thus we can 
call the `std::string` constructor that expects a null-terminated C-style string to copy the contents of. Worth noting this 
an easy trap to fall into:

```
std::string s(0); // Same problem, does not create a std::string
                  // of size one with value char value 0
```

Ironically the following does not work [(see it live)](https://godbolt.org/z/Khvf4h):

```
std::string s;
s=0;  // Ill-formed, ambiguous overload are we passing a char or const char*?
s=65; // Ok, likely will result in string storing character A
```

Assigning zero to a `std::string` does not work, it ends up being an ambiguous overload because `std::string` has an overload to [operator=](https://en.cppreference.com/w/cpp/string/basic_string/operator%3D) that takes *char* and an overload that takes *const char\** and so assigning zero is ambiguous.

One last place where zero is special and that is a declaring pure virtual member functions:

```
struct A{
  virtual void f() = 0; // Zero yet again!
};
```

I am not to happy to report that zero was [used to avoid having to add a new keyword so close to release](https://twitter.com/shafikyaghmour/status/1221599185436200960?s=20). There was nothing technical preventing an alternative such as `pure` or `abstract` other than the difficulty of getting a new keyword accepted so close to release.

So things brings an end to zeros super fun, but it does leave us with one burning question (*How does one turn a meme into an article*) also see
[original meme that inspired this post](https://twitter.com/shafikyaghmour/status/1266393104824717312):


![Is this a](https://pbs.twimg.com/media/EZJrUe2U8AAzTtE?format=jpg&name=medium)

Thank you to those who provided feedback on or reviewed this write-up: Ólafur Waage and Jan Wilmans.

Of course in the end, all errors are the author's.

