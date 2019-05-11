---
layout: post 
categories: [C++, undefined behavior]
title: Exploring Undefined Behavior Using Constexpr 
---

# Exploring undefined behavior using constexpr

We hear a lot about undefined behavior, most probably know we should avoid it. Maybe you have heard about specific types of
undefined behavior, overflow, out of bounds memory access, strict aliasing etc. We can look for articles on undefined 
behavior and there are plenty, talks from several conferences. There are even some great tools for catching
undefined behavior including static analysis (compiler warnings, [clang-tidy](https://clang.llvm.org/extra/clang-tidy/), etc.) 
and dynamic analysis ([UBSan](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html),
[ASan](https://clang.llvm.org/docs/AddressSanitizer.html), etc.) 

Let’s say as a good programmer, we want to understand undefined behavior so we can avoid it and hopefully prevent it and catch
it in code review. We read plenty of articles, watch some videos and learn some tools. Now you go to write some new code and you 
think perhaps what you are doing is undefined behavior but you don’t exactly know the term to look-up or the articles don’t 
address a specific case you are dealing with. We know the tools aren’t perfect and you don’t know enough to go to the 
C or C++ standard and figure it out yourself.

## Constant Expressions

Do we have any ways of exploring undefined behavior that gives us more immediate feedback and we can use as a quick check? Actually we do, we have *constant expressions*. *Constant expressions* in C++ have many restrictions about what is allowed and one of those restrictions is that [undefined behavior is not allowed in a constant expression](https://stackoverflow.com/q/21319413/1708801).

What exactly does the C++ standard says about *constant expressions* and *undefined behavior*. We have to look at sections [\[expr.const\]p4](http://eel.is/c++draft/expr.const#4) and [\[expr.const\]p4.6](http://eel.is/c++draft/expr.const#4.6):

>An expression e is a core constant expression **unless the evaluation of e, following the rules of the abstract
machine (6.8.1), would evaluate one of the following expressions**:
>
> ...
>
>an **operation that would have undefined behavior** as specified in [intro] through [cpp] of this document [ Note: including, for example, signed integer overflow ([expr.prop]), certain pointer arithmetic ([expr.add]), division by zero, or certain shift operations — end note ];

So if we have an operation that would have *undefined behavior* in a context that requires a *constant expression* it would not be valid therefore it is *ill-formed*. This is fancy way of saying that the compiler is required to tell you about it if you violate this rule. In standard talk we would say it must provide a diagnostic, which could be a warning or an error. Currently compilers produce a hard error on ill-formed constant expressions as opposed to a warning.

We have another great tool at our service and that is [godbolt also known as Compiler Explorer](https://godbolt.org/).
Compiler Explorer is an interactive compiler, we can use it to obtain diagnostics for small code quickly. If you make a modifications it updates immediately allowing one to iterate quickly over small changes.

Compiler Explorer, combined with the fact that undefined behavior is ill-formed in a constant expressions, allows us to explore what is and what is not undefined behavior in an interactive and quick manner. Now I know what you are thinking, “Shafik, this sounds too much like having your cake and eating it too”, ok so there are some caveats here.  I mentioned before that not everything is allowed in a constant expression. For example heap allocation, reinterpret_cast etc. so there are classes of undefined behavior we cannot explore e.g. use after free and strict aliasing violations. Will also learned about a couple of exceptions, (*this wouldn't be C++ if there weren't exceptions*). There are still plenty of interesting cases to explore and learn from.

### An Example with Arithmetic Overflow

Let’s start our exploration with undefined behavior in arithmetic operations. If we do a [quick search for "C++ overflow"](https://www.google.com/search?client=firefox-b-1-d&q=C%2B%2B+overflow) we find a mix of information. Some of it does not even mention undefined behavior. Let us see what the compilers tell us is undefined behavior and what is not.

Let’s look at some code:

```cpp
#include <limits>

void f1() {
    unsigned int x1=std::numeric_limits<unsigned int>::max()+1; // Overflow one over the max
    unsigned int x2=0u-1u;                                      // Wrap one below the min
    int y1=std::numeric_limits<int>::max()+1;                   // Overflow one over the max
    int y2=std::numeric_limits<int>::min()-1;                   // Underflow one below the min
}
```

We have both overflow and underflow of *unsigned int* and *signed int*. Is this code okay? Maybe we heard that unsigned overflow is fine but signed is not, is this really accurate?  Ok let's explore using *constant expressions*.

### constexpr

Before we do that, let's take a quick look at *constexpr*. The standard requires that objects declared using the *constexpr* specifier have [literal types](https://en.cppreference.com/w/cpp/named_req/LiteralType), they must be initialized and they must be initialized with a *constant expression*. This is covered in [\[dcl.constexpr\]p9](http://eel.is/c++draft/dcl.constexpr#9):

>A constexpr specifier used in an object declaration declares the object as const. Such an object shall
have literal type and shall be initialized. In any constexpr variable declaration, the full-expression of the
initialization shall be a constant expression (7.7). ...

Now we have all the tools we need to start exploring arithmetic overflow and whether it is undefined behavior or not. Let's add some *constexpr* to our declarations and see what happens:

```cpp
#include <limits>

void f1() {
   constexpr unsigned int x1=
      std::numeric_limits<unsigned int>::max()+1;
    constexpr unsigned int x2=0u-1u;
    constexpr int y1=std::numeric_limits<int>::max()+1;  // Line 7
    constexpr int y2=std::numeric_limits<int>::min()-1;  // Line 8
}
```

Adding the *constexpr* specifier to each declaration now means that the initializer must be a valid *constant expression* and if it is not then it is ill-formed and requires a diagnostic. Unlike *ill-formed no diagnostic required* and *undefined behavior* which don't require a diagnostic. [Going to godbolt](https://godbolt.org/z/0vx_j6) we see the following diagnostics:


```cpp
<source>:7:19: error: constexpr variable 'y1' must be initialized by a constant expression
    constexpr int y1=std::numeric_limits<int>::max()+1;
                  ^  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

<source>:7:53: note: value 2147483648 is outside the range of representable values of type 'int'
    constexpr int y1=std::numeric_limits<int>::max()+1;
                                                    ^

<source>:8:19: error: constexpr variable 'y2' must be initialized by a constant expression
    constexpr int y2=std::numeric_limits<int>::min()-1;
                  ^  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

<source>:8:53: note: value -2147483649 is outside the range of representable values of type 'int'
    constexpr int y2=std::numeric_limits<int>::min()-1;
                                                    ^
```
We see that signed overflow is undefined behavior while unsigned overflow is not. If we want to go to the standard this is covered in [\[expr\]p4](http://eel.is/c++draft/expr#pre-4):

>If during the evaluation of an expression, the result is not mathematically defined or not in the range of representable values for its type, the behavior is undefined. [ Note: Treatment of division by zero, forming a remainder using a zero divisor, and all floating-point exceptions vary among machines, and is sometimes adjustable by a library function.
— end note ]

Further [\[basic.fundamental\]p2](http://eel.is/c++draft/basic.fundamental#2) tells us that unsigned arithmetic is always modulo 2<sup>N</sup>.

Another interesting case in this category is as follows:

```cpp
constexpr int x = std::numeric_limits<int>::min() / -1;
```

If we [goto the live godbolt session](https://godbolt.org/z/Ecp88m) we see is also undefined behavior. Assuming [LP64 data model](https://en.wikipedia.org/wiki/64-bit_computing#64-bit_data_models) we have `-2147483648` which when divided by `-1` theoretically gives us `2147483648` which is not representable by *int*.

## The Benefits

Code evaluated in a context that requires a constant expression should be free of undefined behavior. If your interface is all *constexpr* and you know the range of your inputs you can use *static_assert* to prove your code is free of undefined behavior. Any code that is tested in a constant expression context is free of unconditional undefined behavior e.g. division by zero. 

While not being sufficient to protect us against undefined behavior in all cases it gives us a lot of power to catch undefined behavior if we are willing and able to apply specific constraints. It also provides us a sandbox to experiment with whether an expression invokes undefined behavior or not. It gives us a much smaller surface area of knowledge required. If we can execute the code in a constant expression context then the compiler will flag undefined behavior. We only have to find out why that specific behavior is undefined. As opposed to the much harder problem of knowing enough of the standard to catch all undefined behavior beforehand during code review. 

## Let's explore 

Now that we have some powerful undefined behavior exploration tools, let's explore all the undefined behavior we can catch with these tools. While there may be a lot of undefined behavior not explorable in the constant expression context, what we can explore is rich and most are sure to learn something new and surprising. 

### Conversions and values that can not be represented

Closely related to overflow is the question of what happens when we convert an integral or floating-point type to a smaller sized type? Do we merely obtain the closest number? Do we wrap around in some fashion? Is it undefined behavior? Hopefully at this point you won't be surprised that some cases are undefined behavior, which ones though? [Let's goto to the code](https://godbolt.org/z/2ZfPKt):

```code
 constexpr unsigned int u = std::numeric_limits<unsigned int>::max();  // 1
 constexpr int i = u;                                                  // Line 6
    
 constexpr double d =
     static_cast<double>(std::numeric_limits<int>::max()) + 1;         // 2
 constexpr int x = d;                                                  // Line 10

 constexpr double d2 = std::numeric_limits<double>::max();             // 3
 constexpr float f = d2;                                               // Line 13
```

On lines 6, 10 and 13 we are converting a number to a type in which it cannot be represented. The diagnostics tell us that case `1` is well-defined but case `2` and `3` are not:

```cpp
<source>:10:16: error: constexpr variable 'x' must be initialized by a constant expression
 constexpr int x = d;
               ^   ~

<source>:10:20: note: value 2147483648 is outside the range of representable values of type 'const int'
 constexpr int x = d;
                   ^

<source>:13:18: error: constexpr variable 'f' must be initialized by a constant expression
 constexpr float f = d2;
                 ^   ~~

<source>:13:22: note: value 1.797693134862316E+308 is outside the range of representable values of type 'const float'
 constexpr float f = d2;
                     ^
```

Although GCC does not seem to catch case `3`. 

The standard tells us that case `1` is impelmentation defined, see [\[conv.integral\]p3](https://timsong-cpp.github.io/cppwp/n4659/conv.integral#3) (*this changes in C++20 it modulo 2<sup>N</sup>*):

>If the destination type is signed, the value is unchanged if it can be represented in the destination type; otherwise, the value is implementation-defined.

The standard tells us case `2` and `3` are undefined behavior, see [\[conv.dobule\]p1](https://timsong-cpp.github.io/cppwp/n4659/conv.double#1):

>A prvalue of floating-point type can be converted to a prvalue of another floating-point type. If the source value can be exactly represented in the destination type, the result of the conversion is that exact representation. If the source value is between two adjacent destination values, the result of the conversion is an implementation-defined choice of either of those values. **Otherwise, the behavior is undefined.**

and [\[conv.fpint\]p1](https://timsong-cpp.github.io/cppwp/n4659/conv.fpint#1):

>A prvalue of a floating-point type can be converted to a prvalue of an integer type. The conversion truncates; that is, the fractional part is discarded. **The behavior is undefined if the truncated value cannot be represented in the destination type.** [ Note: If the destination type is bool, see [conv.bool].  — end note ] 

## Division by zero

Now some may know that integer division by zero is undefined behavior, but there is a lot of mixed information over whether floating-point division by zero is also undefined behavior since we do have floating-point values such as `NaN` and `Inf` that seem like reasonable alternatives to undefined behavior. Let's goto the code:

```cpp
constexpr int x = 1/0;        // Line 2
constexpr double d = 1.0/0.0; // Line 3
```

We see [that both versions are undefined behavior](https://godbolt.org/z/tRM6oF):


```cpp
<source>:2:24: error: division by zero is not a constant expression
    constexpr int x = 1/0;
                        ^

<source>:3:28: error: '(1.0e+0 / 0.0)' is not a constant expression
    constexpr double d = 1.0/0.0;
                         ~~~^~~~
```

even though *IEEE 754* does have well defined results for the floating-point case. Not all platforms use *IEEE 754* we can find some of these in the Stack Overflow question [Exotic architectures the standards committees care about](https://stackoverflow.com/q/6971886/1708801).

## Shifty characters

Right after additive operators in the standard, we currently have shift operators and since we are in the neighborhood, why not stop by and see what undefined behavior we can learn about there. What are some cases we might be concerned about? What result should we expect for:

- Shifting greater than the bit-width of the type?
- Shifting by a negative shift? 
- Shifting a negative number?
- Shifting into the sign bit?

```cpp
void foo() {
    static_assert(sizeof(int) == 4 && CHAR_BIT == 8 );
    
    constexpr int y1 = 1 << 32;   // Shifting greater than the bit-width
    constexpr int y2 = 1 >> 32;   // Shifting greater than the bit-width
    constexpr int y3 = 1 << -1;   // Shifting by a negative amount

    constexpr int y4 = -1 << 12;  // Shifting a negative number

    constexpr int y5 = 1 << 31;   // Shifting into the sign bit
}
```

[Going to the code](https://godbolt.org/z/p7onyC) shows us that all but shfting into the sign bit is undefined behavior. The first three are covered by [\[expr.shift\]p1](http://eel.is/c++draft/expr.shift#1):

>The operands shall be of integral or unscoped enumeration type and integral promotions are performed.
The type of the result is that of the promoted left operand.
**The behavior is undefined if the right operand is negative, or greater than or equal to the width of the promoted left operand.**

The fourth case was undefined behavior before C++20, see [\[expr.shift\]p2](https://timsong-cpp.github.io/cppwp/n4659/expr.shift#2):

>... Otherwise, if **E1 has a signed type and non-negative value**, and E1×2<sup>E2</sup> is representable in the corresponding unsigned type of the result type, then that value, converted to the result type, is the resulting value; **otherwise, the behavior is undefined.**

but was made well defined by [p0907r4](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0907r4.html).

## nullptr, everyones favorite pointer 

After arithmetic and shifts I wanted to delve into pointers, specifically `nullptr`. One would think this would be pretty straight forward. Using a `nullptr` is undefined behavior right? Ok, here we have the straight forward case:

```cpp
constexpr int bar() {
    constexpr int* p = nullptr;
    return *p;        // Unconditional UB
}

constexpr void foo() {
    constexpr int x = bar();
}
```

and we should not be surprised that [we obtain plenty of diagnostics for this code](https://godbolt.org/z/cyiVq9):

```cpp
<source>:1:15: error: constexpr function never produces a constant expression [-Winvalid-constexpr]
constexpr int bar() {
              ^

<source>:3:12: note: read of dereferenced null pointer is not allowed in a constant expression
    return *p;        // Unconditional UB
           ^

<source>:7:19: error: constexpr variable 'x' must be initialized by a constant expression
    constexpr int x = bar();
                  ^   ~~~~~

<source>:3:12: note: read of dereferenced null pointer is not allowed in a constant expression
    return *p;        // Unconditional UB
           ^

<source>:7:23: note: in call to 'bar()'
    constexpr int x = bar();
                      ^
```


Ok, what about accessing class members through a `nullptr`, sounds undefined. What about accessing static class members via a `nullptr`? Not sure? Let's goto to the code:

```cpp
struct A{
  constexpr int g() { return 0;}
  constexpr static int f(){ return 1;}
};

static constexpr A* a=nullptr;

void foo() {
  constexpr int x = a->f(); // 1
  constexpr int y = a->g(); // 2
}
```

We have two cases, in case `2` we are accessing a non-static member and case `1` a static member. [Unfortunately we have divergence](https://godbolt.org/z/7NrcFD), Clang says case `2` is undefined behavior but GCC produces no diagnostic. Case `1` is indeed well defined but we need to look at [CWG defect report 315: Is call of static member function through null pointer undefined? ](http://www.open-std.org/jtc1/sc22/wg21/docs/cwg_closed.html#315) which tells us that when accessing a static member via a `nullptr` there is no *lvalue-to-rvalue* conversion. 

## More pointer fun

After `nullptr`, pointer math is the next frontier in our adventures with pointers and undefined behavior. 

### Incrementing pointer out of bounds

Is incrementing a pointer out of bounds undefined behavior? What if we don't use the pointer or dereference it?

```cpp
static const int arrs[10]{};

void foo() {
    constexpr const int* y = arrs + 11;
}
```

Yes, it is undefined behavior which is indicated [by the live code example](https://godbolt.org/z/-E06pt), unfortunately GCC does not catch this case. The one exception is the case of one after the end. This exception is used to [indicate the end of a container or array by std::end for example](https://en.cppreference.com/w/cpp/iterator/end). 


### Incrementing out of bounds but coming back in

So what happens if we increment out of bounds while doing an intermediate calculation but we end up in bounds in the end?

```cpp
constexpr int foo(const int *p) {
    return *((p + 12)-5);
}

constexpr void bar() {
    constexpr int arr[10]{};
    
    constexpr int x = foo(arr);
}
```

Similar to the previous case this is also undefined behavior. [The code](https://godbolt.org/z/D4uayd) and unfortunately in this case gcc also does not diagnose the issue. 

The standard tells us the acceptable bounds for indexing in [\[expr.add\]p4](http://eel.is/c++draft/expr.add#4):

>When an expression J that has integral type is added to or subtracted from an expression P of pointer type, the result has the type of P.
>
>...
>
>- Otherwise, if P points to element x[i] of an array object x with n elements,<sup>80</sup> the expressions P + J and J + P (where J has the value j) point to the (possibly-hypothetical) element x[i+j] if 0≤i+j≤n and the expression P - J points to the (possibly-hypothetical) element x[i−j] if 0≤i−j≤n.
>- Otherwise, the behavior is undefined.

[Footnote 80](http://eel.is/c++draft/expr.add#footnote-80), although non-normative contains the following:

>... A pointer past the last element of an array x of n elements is considered to be equivalent to a pointer to a hypothetical element x[n] for this purpose; see [basic.compound].


This covers both cases of incrementing out of bounds and incrementing out of bounds during an intermediate step. Both cases with the exception of one past the end are undefined behavior. Any addition performed on a pointer must be within bounds or one past the end, which is treated as a *hypothetical* element.

The standard explains that one past the end is a valid pointer in [\[basic.compound\]p3](http://eel.is/c++draft/basic.compound#3):

>...Every value of pointer type is one of the following: 
>
>- a pointer to an object or function (the pointer is said to point to the object or function), or
>- a pointer past the end of an object ([expr.add]), or
>
> ...

### Out of bounds access

Out of bounds access is just the next step in problems after incrementing a pointer out of bounds and is unsurprisingly also undefined behavior. 

```cpp
constexpr int foo(const int *p) {
    return *(p + 12);
}

constexpr void bar() {
    constexpr int arr[10]{};
    
    constexpr int x = foo(arr);
}
```

Unlike with simply creating out of bounds pointers both [clang and gcc catch out of bounds access](https://godbolt.org/z/O2GqaX).

## End of life

Using a variable after its lifetime has ended is undefined behavior but can be hard to catch. There are plenty of ways of creating such situations such as taking and the address or taking a reference and then using that pointer or reference once the variable has gone out of scope or has been released. Memory allocation is not allowed in a constant expression but references are and the example below shows a classic example of returning a reference to a variable that is going out of scope:

```cpp
constexpr int& foo(){
    int x=0;

    return x; // x will soon be out of scope
              // but we return it by reference
} // bye bye x

constexpr int bar() {
    constexpr int x = foo();

    return x;
}
```

If we try this out both [gcc and clang catch this](https://godbolt.org/z/hM607D) although GCC's diagnostic talks about dereferencing a null pointer while Clang tells clearly we are reading a variable whose lifetime had ended. 

Lambdas also offer us another way of accidentally referring to variables after the end of their lifetime, mostly through capture by reference:

```cpp
auto foo(){
    int x=0;
  
    auto f = [&] () {
       return x;
    };

    return f;
}

int bar() {
    auto x = foo();

    return x();
}
```

I wanted to come up with a *constexpr* version of this but it does not seem possible.

## Flowing off the end of a value returning function

Flowing off the end of a value-returning function without using a `return` statement is another undefined behavior that can be hard to catch and compilers do not consistently diagnose it. Below we have a function that returns an *int* has two possible paths but only one *return* statement. If `x` is `0` we will invoke undefined behavior.

```cpp
constexpr int foo(int x) {
   if(x)
      return 1;
   // Oppps we forgot the return 0;
}

void bar(){
    constexpr int x = foo(0);
}
```

Both [clang and gcc catch this case](https://godbolt.org/z/AALpj5) with reasonably clear diagnostics. The standard covers this case in [\[stmt.return\]p2](http://eel.is/c++draft/stmt.return#2.sentence-8):

>...  Flowing off the end of a constructor, a destructor, or a non-coroutine function with a cv void return type is equivalent to a return with no operand. Otherwise, flowing off the end of a function other than main or a coroutine ([dcl.fct.def.coroutine]) results in undefined behavior.

## Modifying a constant object

Attempting to modify a constant object is an undefined behavior that seems harder to invoke by accident but we have many cases where we may want to cast away constantness. One of the most common cases is interacting with a [legacy C API that only take non-const pointers but does not modify the input](https://stackoverflow.com/q/20875520/1708801). In the example below, we cast away the *const* and modify an object that is really a constant:

```cpp
struct B {
  int i;
  double d;
};

constexpr B bar() {
    constexpr B b={10,10.10};
    B *p = const_cast<B*>(&b);

    p->i = 11;
    p->d = 11.11;

   return *p;
}

void foo() {  
   constexpr B y= bar();
}
```

As [the code](https://godbolt.org/z/AYulw0) shows this is undefined behavior, although gcc does not catch this case. This case is covered in [\[dcl.type.cv\]p4](http://eel.is/c++draft/dcl.type.cv#4):

> Except that any class member declared mutable ([dcl.stc]) can be modified, any attempt to modify ([expr.ass], [expr.post.incr], [expr.pre.incr]) a const object ([basic.type.qualifier]) during its lifetime ([basic.life]) results in undefined behavior ...

It is worth seeing that only reading from `p` would be well-defined behavior:

```cpp
struct B {
  int i;
  double d;
};

constexpr B bar() {
   constexpr B b={10,10.10};
   B *p = const_cast<B*>(&b);
    
   int x = p->i;
   
   return *p;
}

void foo() {  
   constexpr B y= bar();
}
```

Looking at [the live example](https://godbolt.org/z/PaADef) we see it is well-formed.

## Accessing a non-active union member

Type punning is a way to interpret an object as a different type. Typically type punning [falls afoul of strict aliasing rules](https://gist.github.com/shafik/848ae25ee209f698763cffee272a58f8) although another common method [involves using the inactive member a union](https://stackoverflow.com/q/25664848/1708801). Which while valid C it is undefined behavior in C++. 

Below, we are attempting to type pun a *float* into an *int*. We initialize `f` as the active member of the union but we then access `k`, which is not the active member, in order to *"reinterpret"* the bits of the *float* as an *int*.

```cpp
union Y { float f; int k; };
void g() {
  constexpr Y y = { 1.0f }; // OK, y.x is active union member (10.3)
  constexpr int n = y.k;    // Line 4
}
```

When we goto [the code](https://godbolt.org/z/0-BxYF), both Clang and GCC say it is ill-formed and clang even provides a pretty solid diagnostic for this case:

```cpp
<source>:4:21: note: read of member 'k' of union with active member 'f' is not allowed in a constant expression
  constexpr int n = y.k;
                    ^
```

The standard tells us only one member of a union can be active and therefore whose lifetime has begun, see [\[class.union\]p1](http://eel.is/c++draft/class.union#1):

>... a non-static data member is active if its name refers to an object whose lifetime has begun and has not ended ([basic.life]). At most one of the non-static data members of an object of union type can be active at any time, that is, the value of at most one of the non-static data members can be stored in a union at any time ...


## Casting int to enum outside its range

This case was inspired by a [poll I ran a while ago](https://twitter.com/shafikyaghmour/status/1034665721286950913) about what values are valid to cast into an enum, which was in turn inspired by [this Twitter question](https://twitter.com/lefticus/status/1027928945604218881). For an enum with a fixed type ([with an enum-base](https://en.cppreference.com/w/cpp/language/enum)) or a [scoped enum](https://en.cppreference.com/w/cpp/language/enum#Scoped_enumerations) the valid values for the enum have the range of the underlying type. Otherwise, the valid values are roughly limited to range required to represent all the enumerators. 

In the example below, *enum A* requires 1 bit to represent the enumerators and because it is not a *scoped enum* nor does it have a *fixed type* then any value requiring more than one bit to represent it is invalid, in this case the value `4` requires two bits.

```cpp
enum A {e1=0, e2};

constexpr int foo() {
    constexpr A a1 = 
       static_cast<A>(4);

       return a1;
}

constexpr int bar() {
    constexpr int x = foo();

    return x;
}

int main() {
    return bar();
}
```

Sadly [no compilers catch this case in a constant expression context](https://godbolt.org/z/ZWI2xb) but [clang does catch a similar case with UBSan](https://wandbox.org/permlink/s6judERNBKYIgspG). The standard covers this in [\[dcl.enum\]p8](http://eel.is/c++draft/dcl.enum#8):

>For an enumeration whose underlying type is fixed, the values of the enumeration are the values of the underlying type.
Otherwise, the values of the enumeration are the values representable by a hypothetical integer type with minimal width M such that all enumerators can be represented. The width of the smallest bit-field large enough to hold all the values of the enumeration type is M. It is possible to define an enumeration that has values not defined by any of its enumerators. ...

and [\[expr.static.cast\]p10](http://eel.is/c++draft/expr.static.cast#10):

>A value of integral or enumeration type can be explicitly converted to a complete enumeration type.
If the enumeration type has a fixed underlying type, the value is first converted to that type by integral conversion, if necessary, and then to the enumeration type. **If the enumeration type does not have a fixed underlying type, the value is unchanged if the original value is within the range of the enumeration values ([dcl.enum]), and otherwise, the behavior is undefined** ...

Also relevant is [defect report 2338](http://www.open-std.org/jtc1/sc22/wg21/docs/cwg_defects.html#2338) which I believe is where the current wording originated.

## Multiple unsequenced modifications

One of the more infamous questions on Stack Overflow: [Why are these constructs using pre and post-increment undefined behavior?](https://stackoverflow.com/q/949433/1708801) which deals with the modifications of a *lvalue* multiple times without a *sequence point* or more modernly referred to as *unsequenced modifications*. It is a questions that in one form or another gets asked frequently usually not realizing it is undefined behavior and looking for a rationale for their specific result or why the result changes on a different compiler.

Below we have a simple version of this problem where `x` is modified twice and there is no sequencing between the operands of the additions operator:

```cpp
constexpr int f(int x) {
  return x++ + x++;
} 
    
int main() { 
  constexpr int x = 2; 
  constexpr int y = f(x);
}
```

If [we goto the code](https://godbolt.org/z/Uai2P9) we sadly see that no compiler diagnoses this case as *ill-formed*. I [tweeted](https://twitter.com/shafikyaghmour/status/1024334313846792193) about this case a while ago and it has been tentatively confirmed as a bug. 

If I understand correctly this may be very hard to catch in a constant expression context. One possible solution to this would be enforcing a strict left to right evaluation order in a constant expression context. This would remove the possibility for undefined but introduce an inconsistency between order of operations is done in and outside of a constant expression context. This would be an unfortunate inconsistency but better than allowing undefined behavior in a constant expression context.

## One More Inconsistency, Guaranteed Copy Elision

In C++17, the circumstances where copy elision is required were strengthened to be guaranteed in more situations; this is called [guaranteed copy elision](https://en.cppreference.com/w/cpp/language/copy_elision). A recent change to the standard says for the [NVRO case](https://www.fluentcpp.com/2016/11/28/return-value-optimizations/) copy elision is not performed in a constant expression context, see [\[class.copy.elision\]p1](http://eel.is/c++draft/class.copy.elision#1):

>...Copy elision is not permitted where an expression is evaluated in a context requiring a constant expression ([expr.const]) and in constant initialization ([basic.start.static]). [ Note: Copy elision might be performed if the same expression is evaluated in another context.— end note ]

To understand why this is the case it is probably best to explore an example shared with me by [Richard Smith](https://twitter.com/zygoloid):

```cpp
struct B {B* self=this;};
extern const B b;
constexpr B f() {
    B b;                              // Line 4
    if(&b == &::b) return B();        // Line 5
    else return b;                    // Line 6
}
constexpr B b=f(); // is b.self == b  // Line 8
```

Take some time to meditate over this example, we need to consider [defect report 2022: Copy elision in constant expressions](http://www.open-std.org/jtc1/sc22/wg21/docs/cwg_defects.html#2022) which required copy elision in a constant expression context:

1. The local variable `b` in `f()` requires copy elision if it is returned via line `6` 
2. If we have copy elision then the `B` in Line `6` would be constructed in place at line `8` 
3. This would mean the condition on line `5` would be *true* that is `&b == &::b`
4. Therefore we won't reach line `6` and there is no copy elision
5. Therefore line `5` is then *false*
6. We have a paradox, the optimization is only performed if we don't perform the optimization.
7. 🤯

So requiring copy elision in a constant expression context in the *NVRO* case allows for these impossible conditions. The current wording is a partial reversal of [defect report 2022: Copy elision in constant expressions](http://www.open-std.org/jtc1/sc22/wg21/docs/cwg_defects.html#2022) which created the original requirement.



## An Example, A Strong Integer Type

Before we wrap up I would like to leave you with an example of how this could be applied. The following example will demonstrate how we could construct a simple strong `int` type that when using in `constexpr` contexts would catch all the common undefined behavior we might run into when working with `int`. 

Strong types for this example gives us the advantage of removing implicit conversions and forces all the constraints to be handled through the type interface. It is really a toy example but will do. Strong types has been covered in great detail in [Jonathan Boccara's series on strong types](https://www.fluentcpp.com/2016/12/08/strong-types-for-strong-interfaces/) and [Bjorn Fahller has a nice talk on the subject](https://www.youtube.com/watch?time_continue=2&v=SWHvNvY-PHw) and we can find more detailed uses there.

Below we have a `Integer` type that wrap an `int` and provides the basic arithmetic operations and shifting:

```cpp
struct Integer {
   constexpr Integer(int v){value = v;}
   constexpr Integer(double d){value = d;}
   constexpr Integer(const Integer&) = default;

   int Value() const {return value;}

   constexpr Integer operator+(Integer y) const { 
       return {value + y.value};
   }

   constexpr Integer operator-(Integer y) const { 
       return {value - y.value};
   }

   constexpr Integer operator*(Integer y) const {
       return {value*y.value};
   }

   constexpr Integer operator/(Integer y) const {
       return {value/y.value};
   }

   constexpr Integer operator<<(Integer shift) const {
       return {value << shift.value};
   }

   constexpr Integer operator>>(Integer shift) const {
       return {value >> shift.value};
   }

   int value{};
};
```

We can see some examples which include several of the undefined behavior we have outlined in the article:

```cpp
  constexpr Integer i_int_max{INT_MAX};
  constexpr Integer i_int_max_plus_one{i_int_max+1}; // Overflow
  constexpr Integer i_one{1};
  constexpr Integer i_zero{0};
  constexpr Integer i_divide_by_zero = i_one/i_zero;  // Divide by zero
  constexpr Integer i_double_max{DBL_MAX}; // double value outside of range representable by int
  constexpr Integer i_int_min{INT_MIN};
  constexpr Integer i_minus_one{-1};
  constexpr Integer i_overflow_division = i_int_min/i_minus_one;  // Overflow
  constexpr Integer i_shift_ub1 = i_one << 32;
  constexpr Integer i_shift_ub2 = i_minus_one << 1;
  constexpr Integer i_shift_ub3 = i_one << -1;
```

The [live code example](https://godbolt.org/z/ScpyN1) shows that we will catch all the undefined behavior in this example. It does require that we use `Integer` in a *constexpr* context in order for this to work. In C++20 though we will [get immediate functions](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1073r3.html) via *consteval* which will require that function be called in a *constant expression* context. 

## Sanitizers

As we come towards the end of the article, I want to mention that although we have some great compile time tools to help catch and explore undefined behavior, not all problems are solved at compile time with *constant expressions*. One of the tools available to catch undefined behavior at runtime are sanitizers. Sanitizers will instrument checks into our code during compilation. The two main sanitizers are [UBSan](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html) and [ASan](https://clang.llvm.org/docs/AddressSanitizer.html). 

## Conclusions

We have learned about *constant expressions* and that *undefined behavior* is forbidden in a *constant expression* context. We have learned that we can leverage this to catch and explore undefined behavior using `constexpr`. We have explored what I believe is most of the undefined behavior that we could possibly invoke in a constant expression context. Hopefully you have learned some new tricks and learned some undefined behavior that we should all be avoiding.

Maybe this will motivate you to write more code at compile time, confident that it will be free of undefined behavior. Maybe the next time you are suspicious about a piece of code you will plug it into a *constexpr* and see if you find undefined behavior.

One major caveat is that we are relying on the compiler to catch undefined behavior for us but compilers do have bugs. We have seen several examples in this posts of inconsistent results. These represent bugs, so we should strive to test with multiple compilers if possible to avoid false negatives as well as false positives.  

Second caveat is *Guaranteed Copy Elision* with respect to *NVRO* and multiple unsequenced mutations where the behavior in one case is already different in the constant expression context and in the case of unsequenced mutations may change to be different. 

Thank you to those who provided feedback on or reviewed this write-up: Patricia Aas, Aaron Ballman, Hana Dusíková, Björn Fahller, Manish Goregaokar, Patrice Roy and Richard Smith.

Of course in the end, all errors are the author's.
