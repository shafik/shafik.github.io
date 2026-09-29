---
layout: post
categories: [C++]
title: The Usual Arithmetic Confusions
---

There are a lot of aspects of C++ that are not well understood and lead to all sorts of confusion. The *usual arithmetic conversions*
and the *integral promotions* are two such aspects. Certain binary operators (arithmetic, relational and spaceship) require
their operands to have a *common type*. The *usual arithmetic conversions* are the set of steps that gets operands to a
*common type*. While the *integral promotions* brings integral types smaller than *int* and *unsigned int* to either *int* or
*unsigned int* depending on which one can represent all the values of the source type. This is one of the areas in C++ that comes directly from C, so pretty much all of these examples applies to C as well as C++.

We will see some examples with results that many may find surprising. After seeing some of these cases we will discuss the rules and how they explain each case. While covering each rule we will present examples to clarify the rule. We will referring to *ILP32* and *LP64* [data models](https://en.wikipedia.org/wiki/64-bit_computing#64-bit_data_models) and it may help to familiarize yourself with the size of types in these models.

It would also help to understand [integer literals](https://en.cppreference.com/w/cpp/language/integer_literal) and the rules
around what the type of an integer literal will be in various cases e.g.

```cpp
// https://cppinsights.io/s/0ffee264
void f() {
  auto x1 = 1;   // Integer literal 1 will have type int
  auto x2 = 1U;  // Integer literal 1U will have type unsigned int
  auto x3 = 1L;  // Integer literal 1L will have type long int
  auto x4 = 1UL; // Integer literal 1UL will have type unsigned long int
}
```

I will be including [C++ Insights](https://cppinsights.io/) links for many examples.

## Some Cursed Examples

Note, that I don't actually expect most to be able to determine the correct results. The examples are meant to be puzzling,
they show some of the worst possible situations. It may be helpful to think about what you would want to
answer to be and why.

Here we have this cute one liner [h/t @johnregehr](https://twitter.com/johnregehr/status/1447340420137054209?s=20) (tweet contains spoilers) which involves a *relational expression* between a *long* value and an *unsigned int* value:

```cpp
std::cout << (-1L < 1U); // What will this output?
```
<details>
  <summary>Click to see answer</summary>
<div class="language-cpp highlighter-rouge"><pre><span class="c1"><span class="c1">//</span> Outputs 1 when using -m64 (LP64) compiler option</span>
<span class="c1"><span class="c1">//</span> Outputs 0 when using -m32 (ILP32) compiler option</span>
<span class="c1"><span class="c1">//</span> Why should the result of a relational operator depend</span>
<span class="c1"><span class="c1">//</span> on the size of long and unsigned int?</span></pre></div>
</details>

We [obtain different output using -m32 Vs -m64 compiler command line options](https://godbolt.org/z/83qfWh3vr).

Our next example is a bit longer but potentially no less puzzling [h/t @cincodenada](https://twitter.com/cincodenada/status/1009590788622200832?s=20)(tweet contains spoilers) a problem which involves subtraction between different sized *unsigned* variables:

```cpp
uint16_t x1 = 1;
uint16_t x2 = 2;
std::cout << x1 - x2 << "\n"; // What will this output?
```
<details>
  <summary>Click to see answer</summary>
<div class="language-cpp highlighter-rouge"><pre><span class="c1"><span class="c1">//</span> Outputs -1</span>
<span class="c1"><span class="c1">//</span> Wait how does subtraction of unsigned types result in a negative number?</span></pre></div>
</details>
<br>


```cpp
uint32_t x3 = 1;
uint32_t x4 = 2;
std::cout << x3 - x4 << "\n"; // What will this output?
```
<details>
  <summary>Click to see answer</summary>
<div class="language-cpp highlighter-rouge"><pre><span class="c1"><span class="c1">//</span> result 4294967295</span>
<span class="c1"><span class="c1">//</span> Expected unsigned wrap around but why not the same result as x1 - x2?</span></pre></div>
</details>
<br>
Here depending on whether we use *uint16_t* or *uint32_t* we obtain different results from subtracting the value `2` from `1`, perhaps even more puzzling is the result in the *uint16_t* case which is negative even though both operands are *unsigned*!

Finally, let's use see this beautiful code [h/t @Myriachan](https://twitter.com/shafikyaghmour/status/1223122542413537286?s=20) involving the multiplication of two *unsigned short* variables:

```cpp
unsigned short x=0xFFFF;
unsigned short y=0xFFFF;
auto z=x*y; // We are multiplying unsigned types so
            // we shouldn't have to worry about undefined behavior
            // Right?
            // What is the result?
```

In this example using `-fsanitize=undefined` provides the following diagnostic [see it live](https://godbolt.org/z/GvoEjoo71):

<details>
  <summary>Click to see answer</summary>
<div class="language-cpp highlighter-rouge"><pre><span class="c1"><span class="c1">//</span> runtime error: signed integer overflow: 65535 * 65535 cannot be represented</span>
<span class="c1"><span class="c1">//</span> in type 'int'</span>
<span class="c1"><span class="c1">//</span> SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior /app/example.cpp:7:13 in</span></pre></div>
</details>


One may be confused at this point, [unsigned overflow is not undefined behavior](https://eel.is/c++draft/basic.fundamental#2.sentence-5),
so how could multiplication of two unsigned integers invoke [signed integer overflow](https://twitter.com/shafikyaghmour/status/1134578146781491201?s=20) which is undefined behavior.

## The Rules

We can find the rules for the Usual Arithmetic Conversions covered in the draft C++ standard section [\[expr.arith.conv\]](https://eel.is/c++draft/expr.arith.conv). The section opens up with paragraph one:

>Many binary operators that expect operands of arithmetic or enumeration type cause conversions and yield result types in a similar way. The purpose is to yield a common type, which is also the type of the result. This pattern is called the usual arithmetic conversions, which are defined as follows:

Let's take a look at each bullet in order. We will see an example for each case.

> - If either operand is of scoped enumeration type, no conversions are performed; if the other operand does not have the same type, the expression is ill-formed.

[Live Example](https://godbolt.org/z/63qncP3Y3):

```cpp
enum class E1 {V1, V2, V3};
enum class E2 {V1, V2, V3};

void f() {
   E1 e1 = E1::V1;
   E1 e2 = E1::V2;

   E2 e3 = E2::V1;
   E2 e4 = E2::V2;

   e1 < e2; // Well-formed both operands have same type
   e1 < e3; // Ill-formed both operands do not have the same type
   e1 < 3;  // Ill-formed both operands do not have the same type
}
```

> - If either operand is of type long double, the other shall be converted to long double.

[Live example](https://cppinsights.io/s/d2892bc6):

```cpp
#include <type_traits>
void f() {
  long double ld = 1;
  int x = 1;

  ld + x; // x will be converted to long double
  static_assert(std::is_same_v<decltype(ld+x),long double>);
}
```

> - Otherwise, if either operand is double, the other shall be converted to double.

[Live example](https://cppinsights.io/s/e5a7094f):

```cpp
#include <type_traits>
void f() {
  double d = 1;
  int x = 1;

  d + x; // x will be converted to double
  static_assert(std::is_same_v<decltype(d+x),double>);
}
```

> - Otherwise, if either operand is float, the other shall be converted to float.

[Live example](https://cppinsights.io/s/a23def46):

```cpp
#include <type_traits>
void f() {
  float f = 1;
  int x = 1;

  f + x; // x will be converted to float
  static_assert(std::is_same_v<decltype(f+x),float>);
}
```

For most the results so far will not seem surprising and is probably what most would expect.
The next bullets deals with *integral promotions* and what happens after the *integral promotions* are applied to each operand. There are several sub-bullets, we will provide an example for each:

> - Otherwise, the integral promotions (7.3.7) shall be performed on both operands.<sup>53</sup> Then the following
rules shall be applied to the promoted operands:

[Live example](https://cppinsights.io/s/3f8a9c5e):

```cpp
#include <type_traits>
void f() {
  int x1 = 1;
  short x2 = 1;

  x1 + x2; // Integer promotions applied and x2 will be promoted to int
  static_assert(std::is_same_v<decltype(x1+x2),int>);

  unsigned int x3 = 1;
  unsigned short x4 = 1;

  x3 + x4; // Integer promotions applied and x4 will be promoted to int
           // x4 will then be converted to unsigned int due to rule two
           // bullets further down
  static_assert(std::is_same_v<decltype(x3+x4),unsigned int>);

  unsigned short x5=0xFFFF;
  unsigned short x6=0xFFFF;
  auto z=x5*x6; // Integer promotions applied and x5 and x6 will be promoted to int
                // Result will be larger than std::numeric_limits<int>::max() and
                // will have signed integer overflow which is undefined behavior
  static_assert(std::is_same_v<decltype(x5*x6),int>);
}
```

>   - If both operands have the same type, no further conversion is needed.

[Live example](https://cppinsights.io/s/de677cd9):

```cpp
#include <type_traits>
void f() {
  char x1 = 1;
  short x2 = 1;

   x1 + x2; // After integer promotions both operands promoted to int
            // Both have the same type no conversion needed
  static_assert(std::is_same_v<decltype(x1+x2),int>);
}
```

>    -   Otherwise, if both operands have signed integer types or both have unsigned integer types, the
operand with the type of lesser integer conversion rank shall be converted to the type of the
operand with greater rank.

[Live example](https://cppinsights.io/s/22d1d58c):

```cpp
#include <type_traits>
void f() {
  int x1 = 1;
  long x2 = 1;

   x1 + x2; // After integer promotions x1 remains int and x2 remains long
            // Both signed, int has rank less than
            // long so x1 will be converted to long
   static_assert(std::is_same_v<decltype(x1+x2),long>);

  unsigned int x3 = 1;
  unsigned long x4 = 1;

  x3 + x4; // After integer promotions x3 remains unsigned int,
           // x4 remains unsigned long
           // Both unsigned, unsigned int has rank less than
           // unsigned long so x3 will be converted to unsigned long
  static_assert(std::is_same_v<decltype(x3+x4),unsigned long>);
}
```

>   - Otherwise, if the operand that has unsigned integer type has rank greater than or equal to the
rank of the type of the other operand, the operand with signed integer type shall be converted to
the type of the operand with unsigned integer type.

[Live example](https://cppinsights.io/s/eaf11751):

```cpp
void f() {
  if (1U > -1) // Integer promotions does not affect the types of the operands
               // unsigned int has rank greater than or equal to int
               // so the right operand will be converted to unsigned int
               // So we have 1U > -1U which is equivalent to
	       // 1U > 4294967295U which is false.
     std::cout << "Greater\n";
  else
     std::cout << "Not greater\n";
}
```

>   - Otherwise, if the type of the operand with signed integer type can represent all of the values of
the type of the operand with unsigned integer type, the operand with unsigned integer type shall
be converted to the type of the operand with signed integer type.
>    - Otherwise, both operands shall be converted to the unsigned integer type corresponding to the
type of the operand with signed integer type.

```cpp
std::cout << (-1L < 1U); // Integer promotions does not affect the types of the
                         // operands
			 //
                         // On and LP64 system, long can represent all the values of
                         // unsigned int so 1U will be converted to long
                         // So we have -1L < 1L which is true
                         //
                         // On and ILP32 system, long can not represent all the
			 // values of unsigned int So unsigned int and long will be
                         // converted to unsigned long which is the unsigned integer
                         // type corresponding to the type of signed operand (long).
			 //
                         // So we have -1UL < 1UL which is equivalent to
			 // 4294967295UL < 1UL which is false.
                         //
                         // See https://stackoverflow.com/a/22801135/1708801 to
			 // understand why -1UL becomes 4294967295UL
```

## Conclusions

We learned about the *usual arithmetic conversions* and how they bring the operands of many binary operators to a
common type before evaluation. We saw several examples and in most cases the examples actually did what we would have
expected. We also saw a few cases that
had unintuitive results. We have also seen how the results can vary based on the platform we are on, so your expectations
may be correct on one platform but incorrect on another.

Unfortunately even experts can easily get mixed up when it comes to the *usual arithmetic conversions* but some things to keep in mind which may help:

- When we mix *signed* and *unsigned* types a conversion to one of these types will be required. Deciding what conversion makes sense beforehand and apply the conversion explicitly may help prevent confusion.
- When dealing with *integral* types smaller then *int* and *unsigned int* the *integral promotions* will require a conversion to a larger type. These can lead to changes in *sign*. It may make sense to apply the conversion explicitly.
- We saw that mixing *signed* and *unsigned* types in *relational* expressions can lead to unintuitive results due to changes in *sign*.
- We need to understand the valid range of values of our operands.
  - Arithmetic on *unsigned* values never leads to undefined behavior but *unsigned overflow* may not be the behavior you want.
  - Arithmetic on *signed* values can lead to *undefined behavior* if the result overflows.
- [C++ Insights](https://cppinsights.io/) can be useful in seeing the conversions applied by the compiler due to *usual arithmetic conversions* and *integral promotions*.

I did not provide a detailed look into how the *integral promotions* work, this is a topic that deserves its own article. For the more curious you can read up about the *integral promotions* in the draft C++ standard section [\[conv.prom\]](https://eel.is/c++draft/conv.prom)
