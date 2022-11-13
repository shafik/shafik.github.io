---
layout: post
categories: [C++]
title: Enums In C++, Choice is Oft Beguiled 
---

## The Basics

In C++ we have several different options for *enum* types. These different choices affect the scope the *enumerators* are available in, the range of underlying values we can use and even whether setting certain values to an *enum* invokes *undefined behavior*. We are going to explore the different choices available and the consequences of those choices. I will follow this post with a second one exploring some of clang's implementation details, undefined behavior, UBSan, and constant expressions.

In C++11 forward we have three different types of enums. We have C-style plain *enum*, *enum* with explicit underlying type
specified using an *enum-base* and *scoped enums* see [\[dcl.enum\]p5](https://eel.is/c++draft/enum#dcl.enum-5), ([live code example](https://godbolt.org/z/MhKzvPzTc)):

```cpp
enum PlainEnum {};
enum FixedUnderlyingTypeEnum : int {};
                            // ^^^ enum-base
enum class ScopedEnum {};
enum struct AlsoScopedEnum {};
```

The *enumerators* of both plain *enums* and an *enum* with an *enum-base* are *unscoped enumerators* and they are available in the scope that the *enum* is declared in. Folks often say the enumerators *leak*, and this can be undesirable in many cases such as in the global scope where unintentional shadowing may be hard to spot. While for a *scoped enum* we are required to use a qualified name in order to access the *enumerators* ([see it live](https://godbolt.org/z/xETE3G1G1)):

```cpp
enum E {
  E1,E2
};

enum class EC {
  EC1, EC2
};

int main() {
   E e1 = E1;       // Unscoped enumerator available
                    // in the scope of E is declared in
   EC e2 = EC::EC1; // Scoped enumerator require qualified name
}
```

*Unscoped enumerators* can also implicitly convert to integer types. This can easily lead to unintended comparisons, although a high quality compiler should produce a diagnostic for this case ([see live example](https://godbolt.org/z/7G3es4K34)):

```cpp
enum Card {jack, king, queen, ace};
enum Color : int {red, black};

int main() {
  Card card{};
  int x{};

  if(card == red) {}     // Implicit conversion to integer
                         // Comparing a Card and a Color
  if(x == Color::red) {} // Implicit conversion to integer
                         // Comparing an int and a Color
}
```

## Complete or not a Complete Type

A *scoped enum* is a complete type once the base type (if any) is seen. While an *enum* without a fixed *underlying type* is not a complete type until the closing brace of the enum declaration. If we don't have a fixed underlying type we need to see the range of the enumerators before we can determine what the underlying type will be, see [\[dcl.enum\]p6](https://eel.is/c++draft/enum#dcl.enum-6) ([Live code example](https://godbolt.org/z/9vo8cTY3K))

```cpp
enum PlainEnum {
  PE1,
  PE2 = (PlainEnum)0 // Ill-formed, PlainEnum is not a complete type
};

enum FixedUnderlyingTypeEnum : int {
  FE1,
  FE2 = (FixedUnderlyingTypeEnum)0 // ok FixedUnderlyingTypeEnum is a complete type
};

enum class ScopedEnum {
  SE1,
  SE2=(int)(ScopedEnum)(1) // ok ScopedEnum is a complete type
};
```

For an *enum* with a *base-type* the *underlying type* of the *enumerators* is the *base-type*. For a *scoped enum* the *underlying type* is *int*. A consequence of this is that the value of each enumerator must fit into the *underlying type* ([see it live](https://godbolt.org/z/3aWro7PMY)):

```cpp
enum FixedUnderlyingTypeEnum : char {
  FE1,          // underlying type is char
  FE2 = INT_MAX // Ill-formed INT_MAX too large for underlying type
};

enum class ScopedEnum {
  SE1,           // Underlying type int
  SE2 = UINT_MAX // Ill-formed UINT_MAX too large for underlying type
};
```


## Determining the Underlying Type

How do we determine the *underlying type* of the *enum* whose type is not fixed? Before the closing brace the *underlying type* for each *enumerator* depends on the initializers if any, see [\[dcl.enum\]p5](https://eel.is/c++draft/enum#dcl.enum-5). If there is no initializer then the value of *enumerator* is zero if it is the first or the incremented value of the previous *enumerator*. After the closing brace, it is the type that can represent all the enumerators, if such a type exists, see [\[dcl.enum\]p7](https://eel.is/c++draft/enum#dcl.enum-7) and [\[dcl.enum\]p8](https://eel.is/c++draft/enum#dcl.enum-8). Let's see some examples [live code](https://godbolt.org/z/71eGvvcGc):

```cpp
// Assuming LP64
enum E {
  E1, // unspecified signed integer type
  E2 = UINT_MAX-1, // Unsigned int since that is the type of the expression
  E3, // unsigned int since the previous type is unsigned int and the incremented
      // value fits in unsigned int
  E4, // unspecified integral type since UINT_MAX + 2 does not fit into unsigned int
  E5 = ULLONG_MAX, // unsigned long long
  E6 // ill-formed, does not fit into largest integer type
};

enum E2 {
  E21 = LONG_MIN,
  E22 = ULONG_MAX // Ill-formed no underlying type can represent both LONG_MIN and ULONG_MAX
};

enum E3 {}; // as-if there was a single enumerator with value 0
```


## Range of values

For enums with a fixed underlying type, the range of values of the enum are the full range of the underlying type, see [\[dcl.enum\]p8](https://eel.is/c++draft/enum#dcl.enum-8) which says:

>For an enumeration whose underlying type is fixed, the values of the enumeration are the values of the underlying type. ...

That means for an enum with a fixed underlying type it is valid to set it to a value outside of the *enumerators*. This is necessary to allow various idioms such as using enum as bit flags to work properly ([see it live](https://godbolt.org/z/Kz6q9fove)):

```cpp
enum EBitFlags {
  Flag1 = 1 << 0,
  Flag2 = 1 << 1,
  Flag3 = 1 << 2,
  Flag4 = 1 << 3
};

void f() {
    EBitFlags ef = static_cast<EBitFlags>(Flag1 | Flag3); // ef will have a value of 5
                                                          // which is not of one of the enumerators
}
```

[std::byte](https://en.cppreference.com/w/cpp/types/byte) which is a *scoped enum* with an underlying type of *unsigned char* takes advantage of this:

```cpp
enum class byte : unsigned char {} ; // enum with no enumerators but since underlying type is unsigned char
                                     // it can take on all the values of this type
```

For enums without a fixed underlying type the range of values it can take is based on the number of bits required to represent the enumerators. This is covered in [\[del.enum\]p8](https://eel.is/c++draft/enum#dcl.enum-8) which says:

>Otherwise, the values of the enumeration are the values representable by a hypothetical integer type with minimal width M such that all enumerators can be represented. The width of the smallest bit-field large enough to hold all the values of the enumeration type is M.

We can work through a few examples ([see it live](https://godbolt.org/z/P84zPbbjY)):

```cpp
enum E1 {e1=0};             // Range of values [0,1]
enum EEmpty {};             // Range of values [0,1]
```

We only need one bit to represent `0` and that also allows us to represent `1` since we consider the underlying type unsigned since it has no negative values. Since an *enum* with no *enumerators* behaves as-if it had a single *enumerator* with value `0` then this case matches the `E1` case. The next example, we also have an enumerator with a negative value, we need four bits but because of how [Two's Complement](https://www.cs.cornell.edu/~tomf/notes/cps104/twoscomp.html) works we have an asymmetric range:

```cpp
enum E2 {e21=-4, e22=4};    // Range of values [-8,7]
```

The next few examples demonstrates more cases but they basically follow the previous two:

```cpp
enum E3 {e31=0, e32=4};     // Range of values [0,7]
enum E4 {e41=-4, e42=1024}; // Range of values [-2048,2047]
enum EMax {em1=-1, em2=__INT_MAX__}; // Range of values [-2147483648, 2147483647]
```

## Using enum

One last feature to discuss is `using enum`. This feature allows us to introduce enumerators of a *scoped enum* into a scope of our choice. While often we want to prevent *enumerators* from leaking into scopes sometimes we want to purposefully allow it. For example it could make *case labels* in a *switch* less verbose. I will use an example [straight from the proposal for this feature](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1099r5.html):

```cpp
enum class rgba_color_channel { red, green, blue, alpha };

// Without using enum, the case labels are way more verbose than many would like
std::string_view to_string(rgba_color_channel channel) {
  switch (channel) {
    case rgba_color_channel::red:   return "red";
    case rgba_color_channel::green: return "green";
    case rgba_color_channel::blue:  return "blue";
    case rgba_color_channel::alpha: return "alpha";
  }
}

// With using enum
std::string_view to_string(rgba_color_channel channel) {
  switch (my_channel) {
    using enum rgba_color_channel;
    case red:   return "red";
    case green: return "green";
    case blue:  return "blue";
    case alpha: return "alpha";
  }
}
```

Thank you to those who provided feedback on or reviewed this write-up: Tom Honermann and Rosa Yaghmour.

Of course in the end, all errors are the author's.
