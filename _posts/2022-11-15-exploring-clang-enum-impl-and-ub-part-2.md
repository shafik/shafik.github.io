---
layout: post
categories: [C++]
title: Exploring Clangs Enum implementation and How we Catch Undefined Behavior 
---

## Intro

In [Part 1: Enums In C++, Choice is Oft Beguiled](https://shafik.github.io/c++/2022/11/12/enums-in-cpp-choice-oft-beguilded-part-1.html) I talked about how enums work in C++ and the various choices we have. If you are not familiar with how enums work reading part 1 will help you to understand part 2.

In this part we will dive into some of the clang internals for tracking the range of values of an *enum* and how it is used by `UBSan` and the optimizer. Finally we will look at *constant expressions* and a recent set of patches I landed to diagnose *undefined behavior*.

## Keeping Track of Positive and Negative Bits

In clang for an enum without a fixed underlying type we need to track the range of values it can represent. In the *AST* we represent an enum using an `EnumDecl` which *is a* `DeclContext` and within the `DeclContext` we have `NumPositiveBits` and `NumNegativeBits`. These fields are part of the class `EnumDeclBitfields` which is part of *anonymous union* used as a *Sum Type* to represent the various bits used by the various *derived classes* of `DeclContext`:

```cpp
 class EnumDeclBitfields {

    ...

    /// Width in bits required to store all the non-negative
    /// enumerators of this enum.
    uint64_t NumPositiveBits : 8;

    /// Width in bits required to store all the negative
    /// enumerators of this enum.
    uint64_t NumNegativeBits : 8;

    ...
```

When we are processing the body of the *enum* in `Sema::ActOnEnumBody(...)` we use the following code to calculate the positive and negative bits:

```cpp
  unsigned NumNegativeBits = 0;
  unsigned NumPositiveBits = 0;

  for (unsigned i = 0, e = Elements.size(); i != e; ++i) {
    EnumConstantDecl *ECD =
      cast_or_null<EnumConstantDecl>(Elements[i]);
    if (!ECD) continue;  // Already issued a diagnostic.

    const llvm::APSInt &InitVal = ECD->getInitVal();

    // Keep track of the size of positive and negative values.
    if (InitVal.isUnsigned() || InitVal.isNonNegative()) {
      // If the enumerator is zero that should still be counted as a positive
      // bit since we need a bit to store the value zero.
      unsigned ActiveBits = InitVal.getActiveBits();
      NumPositiveBits = std::max({NumPositiveBits, ActiveBits, 1u});
    } else {
      NumNegativeBits = std::max(NumNegativeBits,
                                 (unsigned)InitVal.getMinSignedBits());
    }
  }

  // If we have have an empty set of enumerators we still need one bit.
  // From [dcl.enum]p8
  // If the enumerator-list is empty, the values of the enumeration are as if
  // the enumeration had a single enumerator with value 0
  if (!NumPositiveBits && !NumNegativeBits)
    NumPositiveBits = 1;
```

For each enumerator we will calculate the highest bit required to represent the *enumerator*. If it is positive value we will update `NumPositiveBits` and if it is negative value we will update `NumNegativeBits`. At the end if we have no *enumerators* it will treat it as-if we had a single enumerator of value `0` and set `NumPositiveBits` to `1`.


## UBSan

Since we know the number of positive and negative bits required to represent all the enumerators we can calculate the
range of underlying values for an *enum* without a fixed underlying type. `EnumDecl` provides a member function `getValueRange(...)` which calculate the maximum and minimum values inclusive:

```cpp
void EnumDecl::getValueRange(llvm::APInt &Max, llvm::APInt &Min) const {
  unsigned Bitwidth = getASTContext().getIntWidth(getIntegerType());
  unsigned NumNegativeBits = getNumNegativeBits();
  unsigned NumPositiveBits = getNumPositiveBits();

  if (NumNegativeBits) {
    unsigned NumBits = std::max(NumNegativeBits, NumPositiveBits + 1);
    Max = llvm::APInt(Bitwidth, 1) << (NumBits - 1);
    Min = -Max;
  } else {
    Max = llvm::APInt(Bitwidth, 1) << NumPositiveBits;
    Min = llvm::APInt::getZero(Bitwidth);
  }
}
```

This is important to be able to calculate this range because if we use `static_cast` to convert an *integral* or *enumeration* type to an *enumeration* type that does not have a fixed underlying type then the resulting value must be within the range of values of the *enumeration*, otherwise it invokes *undefined behavior*, see [\[expr.static.cast\]p10](https://eel.is/c++draft/expr.static.cast#10) which says:

> If the enumeration type does not have a fixed underlying type, the value is unchanged, if the original value is within the range of the enumeration values ([dcl.enum]), and otherwise, the behavior is undefined

For example:

```cpp
enum E1 {e1=0};             // Range of values [0,1]

void f() {
  E1 x = static_cast<E1>(2); // undefined behavior, 2 is outside the range of values
}
```

We can catch this using [UBSan](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html) which will detect the *undefined behavior* when loading the value of the *enumeration* ([see it live](https://godbolt.org/z/qhs9W6GYq)):

```cpp
enum E1 {e1=0};             // Range of values [0,1]

int main() {
  E1 x = static_cast<E1>(2); // undefined behavior, 2 is outside the range of values
  return x; // UBSan will catch that we are loading a value outside the range
}
```

`UBSan` uses `getValueRange(...)` through `getRangeForType(...)` to generate [icmp](https://llvm.org/docs/LangRef.html#icmp-instruction) bound checks, for example ([see it live](https://godbolt.org/z/9hTM4jGrT)):

```cpp
enum E1 {e1=-1,e2=1};             // Range of values [-2,1]

int main() {
  E1 e = static_cast<E1>(8);
  return e; // When -fsanitize=enum is enabled the load of value
            // will cause range check to be generated
}
```

Results in the following IR to be generated to enable checking the enum value is valid:

```llvm
%1 = icmp sle i32 %0, 1, !dbg !23, !nosanitize !20  ; Check max bound
%2 = icmp sge i32 %0, -2, !dbg !23, !nosanitize !20 ; Check min bound
%3 = and i1 %1, %2, !dbg !23, !nosanitize !20       ; Combine checks into a single boolean value
                                                    ; Next if checks fails branch to invalid value handler
br i1 %3, label %cont, label %handler.load_invalid_value, !dbg !23, !prof !24, !nosanitize !20
```

## LLVM Range Info

The *enum* range information is can also be used by the optimizer. If we look at the LLVM language reference for [range meta data](https://llvm.org/docs/LangRef.html#range-metadata) it says:

> range metadata may be attached only to load, call and invoke of integer types. It expresses the possible ranges the loaded value or the value returned by the called function at this call site is in. If the loaded or returned value is not in the specified range, the behavior is undefined.

If we take the previous example and use `-fstrict-enums` combined with optimization level `-O1` or greater during code generation *range meta data* will be attached to the load of the *enum* values. We need to also use `-Xclang -disable-llvm-passes` in order to preserve the IR we want to see for this case ([see it live](https://godbolt.org/z/Gdvah4TGK)):

```llvm
; Range meta data from !28 applies to this load
%0 = load i32, ptr %e, align 4, !dbg !27, !tbaa !23, !range !28
;; emitted code
; Range meta data tells us the range of values [2,2) or [2,1]
!28 = !{i32 -2, i32 2}
```

## Constant Expression

Operations that invoke undefined behavior in a [constant expression context are ill-formed](https://shafik.github.io/c++/undefined%20behavior/2019/05/11/explporing_undefined_behavior_using_constexpr.html). So just as UBSan will catch assigning a value to *enum* without a fixed *underlying type* outside of it's range we should also expect this to be caught in a *constant expression* context. While UBSan requires a load of the value before we can catch this specific *undefined behavior* in a constant expression context we will catch this once the value is assigned ([see it live](https://godbolt.org/z/crYzrPKjK)):

```cpp
enum E1 {e1=0};             // Range of values [0,1]

void f() {
  constexpr E1 x = static_cast<E1>(2); // Ill-formed, invoking undefined behavior is not allowed in
                                       // a constant expression
}
```

Unfortunately [clang was not conforming in this case](https://github.com/llvm/llvm-project/issues/50055). I recently worked on a [fix for this](https://reviews.llvm.org/D130058). The change ended up being too disruptive, so we allowed for a transition period where it could be [turned into a warning](https://reviews.llvm.org/D131307) while fixes were put in place.

We can see from several examples such as [this case with a bit-field type](https://bugs.chromium.org/p/chromium/issues/detail?id=1348574#c7) or this [case Boost MPL](https://github.com/boostorg/mpl/issues/69) where it caught real bugs that had gone unnoticed.

Thank you to those who provided feedback on or reviewed this write-up: Tom Honermann and Rosa Yaghmour.

Of course in the end, all errors are the author's.
