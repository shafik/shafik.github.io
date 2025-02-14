---
layout: post
categories: [C++,undefined behavior,LLVM]
title: What You Need to Know when Optimizations Changes the Behavior of Your C++ 😱 
---

It is often said by those who have been around C++ for a long time that:


*"If the behavior of your program changes when using some level of optimization, then it is likely you have undefined behavior".*

This is not to say that there are no compiler bugs
they do happen but chances are what you are seeing is likely not a compiler bug. While this is all true, there is a catch
here. If someone suspects they may have undefined behavior, how do they find it? It feels like one of those punchlines to a
joke, "Sure everything you told me is true but I am no better off than I was before I asked!".

It is a great question and sadly, I don't have one simple answer to catch them all. What I can do is give you a set of
approaches that for a good number of cases should help you prove or get you closer to eliminating the possibility of
*undefined behaviors* being the cause of your problem.

Note, this can also happen between compilers. One compiler with optimization provides the consistent results but another
compilers results change with optimization level. The following techniques should work in these cases as well.

## Reduce that Bug (Or That One Little Trick)

Before we get started, I would like to point you to a previous article I have written [Triaging clang C++ frontend bugs](https://shafik.github.io/c++/llvm/2024/10/17/triaging-clang-fronend-bugs.html).
In particular I would like you to read the *Do We Need a Reduction?* section. The tools below can will cast a pretty wide net
and may hit other bugs before hitting the one you at the moment care about. A reduction should prevent you tripping over other
issues inadvertently. Once your able to obtain a minimal test case it will make your analysis and possible future bug report
way easier and more effective. The first step of a compiler bug report will almost always be to ask for a minimal test case
if one is not already provided.

## Sanitizers

The potentially easier path is, you have a undefined behavior (*UB*) that is caught by one of the several sanitizers
available. Maybe compiler provide sanitizers including: [clang](https://clang.llvm.org/docs/), [gcc](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html)
and MSVC has [ASan](https://learn.microsoft.com/en-us/cpp/sanitizers/asan?view=msvc-170). For example, with gcc and clang we
can enable *undefined behavior* and *address sanitizers* for both using `-fsanitize=undefined,address`. These *sanitizers*
will catch many *UBs* and memory violation errors. For example say we have the following [code](https://godbolt.org/z/TjzsT96v4):

```cpp
int f() {
   for (int i = 0; i >=0; i++ ){} // Signed integer overflow once passed MAX_INT
   return 1;
}

int main() {
    return f();
}
```

at `-03` both clang:

```asm
f():

main:
```

and gcc:

```asm
f():
.L2:
        jmp     .L2
main:
.L5:
        jmp     .L5
```

generate (for their specific versions) what for most would be pretty surprising code. That is because the code is invoking the
*signed integer overflow undefined behavior* but luckily for this case *UBSan* catches this case easily [see here](https://godbolt.org/z/KqGh8E4b8):

```console
/app/example.cpp:2:28: runtime error: signed integer overflow: 2147483647 + 1 cannot be represented in type 'int'
SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior /app/example.cpp:2:28
```

Data races are another form of *undefined behavior* and [TSan](https://clang.llvm.org/docs/ThreadSanitizer.html) can help
catch those.

Using uninitialized memory can be detected via [MSan](https://clang.llvm.org/docs/MemorySanitizer.html) but it has the one
downside that you have to "recompiled from source, including all dependent libraries", which may not be feasible for many
projects. An alternative to `MSan` is to use `Valgrind`.

So very often, merely running your code with a sanitizer will catch your undefined behavior. It is good practice to run
sanitizers on your code as part of testing if you have the resources to do so continuously but if not at least regularly. They
will catch real bugs and save you painful debugging sessions in the future.

Unfortunately they do not catch all the possible undefined behaviors and so they can not be your sole goto tool.

### An Aside on Signed Overflow

Since we used *singed integer overflow* as an example above it is worth pointing out that clang and gcc have `-fwrapv` flag
which is an extension that makes *signed integer wrapping* well defined. We can see taking the original example and adding
[the -fwrapv flag](https://godbolt.org/z/7bjszT3sd) we obtain good codegen again e.g.:

```cpp
f():
        mov     eax, 1
        ret

main:
        mov     eax, 1
        ret
```

Also note that, with this flag, *UBSan* will no long [treat signed overflow as UB](https://godbolt.org/z/f7sc8d95z).

## Strict Aliasing Violations

A common *UB* violations that can lead to different results under optimization are *strict aliasing* violations. I wrote
[What is the Strict Aliasing Rule and Why do we care?](https://gist.github.com/shafik/848ae25ee209f698763cffee272a58f8) a
while ago and it goes into fine details on the subject.

From a practical perspective using `-fno-strict-aliasing` is usually a quick way to determine if the change in behavior is due
to optimization assumption made around strict aliasing, this [bug report](https://github.com/llvm/llvm-project/issues/68655)
is one example. If the change in behavior goes away once you use `-fno-strict-aliasing` then you almost surely have a strict
aliasing violation.

[Type sanitizer](https://clang.llvm.org/docs/TypeSanitizer.html) is also a useful tool here but it is relatively new and still
has some bugs. So it may not currently catch all cases. I am confident with some more work we can use `TYSan` in the future
instead of having to resort to the `-fno-strict-aliasing` trick.

## Aside on Flags

Some may ask the completely valid question, "well since we have flags that make UB well defined then why not just use them?".
This is a good question and many projects do indeed choose to go with that strategy. Some downsides is that you may be leaving
performance on the table, you will need to measure these effects for your particular project to determine how large the effect
is and whether it is worth it.

Portability is a second consideration. Both clang and gcc strive to be compatible on a *best effort* basis in these areas.
Many extensions are not portable and using these various flags can be a problem in projects that need to be 💯 portable.
So choices one makes today can limit you later on or require larger refactors down the road.

There is also the philosophical perspective, we should never invoke *undefined behavior* if we don't have to, it is just poor
hygiene. Almost all the *undefined behavior* have well defined alternative idioms available that do not leave performance on
the table. Many projects choose to use these flags because the *undefined behavior* is already spread far and wide in their
code base and refactoring may not be practical at that point in time or ever. If you are not in that situation refactoring
now and going with a well defined idiom is usually the better option. At the end of the day, merely understanding the
trade-offs will put you in a better place.

## Constexpr

A while ago I wrote an article [Exploring Undefined Behavior Using Constexpr](https://shafik.github.io/c++/undefined%20behavior/2019/05/11/explporing_undefined_behavior_using_constexpr.html).
The ideas here is that *undefined behavior* is forbidden in *constant expressions* and so will obtain a diagnostic for any
*UB* in your code evaluated in a *constant expression* context. There are constructs not allowed in *constant evaluation* but
now a days the numbers of things outright forbidden has shrunk a lot. If you were able to reduce your problem to a minimal
test case then you may be in a good position to rewrite your code to be *constexpr* friendly.

If we modify our original integer overflow example a bit, so that we avoid *constant evaluation* iteration limits:

```cpp
constexpr int f() {
   for (int i = 0; i >= 0; i = i + 10000 ){}
   return 1;
}

int main() {
    constexpr int x = f();
    return x;
}
```

We will see that the [compilers](https://godbolt.org/z/f6aEacf9e) flag the *signed integer overflow*:

```console
<source>:7:24:   in 'constexpr' expansion of 'f()'
<source>:2:33: error: overflow in constant expression [-fpermissive]
    2 |    for (int i = 0; i >=0; i = i + 10000 ){}
      |                               ~~^~~~~~~
```

Another case we can catch using this methods that other catch us using the inactive member of a *union*. Often referred to
as type punning (*see string aliasing reference above*), the following is an example of punning a *float* value to an *int*
value via a *union* [live example](https://godbolt.org/z/n3MYf4McM):

```cpp
union U {
    float x;
    int y;
};

int f() {
   U u{1.f};

   return u.y; // Type punning float to int, UB!
}

int main() {
    return f();
}
```

If we attempt to do this in a *constant expression* it fails to compile [see it live](https://godbolt.org/z/hfjKM76ao):

```cpp
union U {
    float x;
    int y;
};

constexpr int f() {
   U u{1.f};

   return u.y; // UB read of inative member
}

int main() {
    constexpr int x = f(); // Ill-formed
    return x;
}
```

### An Aside on reinterpret_cast

*constexpr* has come a long way since *C++11* and there is a lot more available to be used in *constexpr* since *C++11*, one
of the major features not available is `reinterpret_cast`. If converting your minimal reproducer gets stuck because of
`reinterpret_cast` that likely points toward your culprit. It is very easy to misuse `reinterpret_cast` in ways that invoke
*undefined behavior*. [Unspecified results](https://eel.is/c++draft/expr.rel#5) of *relational operators* is another area
where dragons may exist.

## Try Different Compiler

Diagnostics can vary widely between compilers and so if it is feasible trying your code across different compilers. One
compiler may not be able to catch the *UB* but another may be able to diagnose it. One of many benefits of writing portable
code is the ability to leverage multiple compilers.

## Asking for Help

Now, what happens when you try all of the tools and they don't find any issues? Does that mean you are undefined behavior
free? Maybe, it could be that you hitting an *undefined behavior* not caught by the tools. It could be that although rare, you
hit a compiler or optimization bug. At this point if you have a minimal test case you potentially have options available
to ask folks for help. The most obvious would be [Stackoverflow](https://stackoverflow.com/), there are a large number of
C++ experts there. [include<C++>](https://www.includecpp.org/) has a discord server that is open to folks, it has a channel
for beginners as well. There are plenty of other options but there are the two I would recommend.

## File a Bug Report

At this point, if you have exhausted all your avenues of investigation and don't have an answer you should definitely file a
bug report against the compiler. Usually the compiler team will be able to identify *undefined behavior* Vs compiler
or optimization bug pretty quickly. All the information from the previous steps are immensely useful to a bug report, they
will be greatly appreciated by those triaging the bug and will usually get you a quicker answer.

## Libc++ Hardening, clang-tidy and Clang Static Analyzer

Since we are talking about bug finding already, there are a whole host of tools out there and it is good to know what is
available. The more tools you run automatically as part of your development process the cleaner code you will have over time.
Here are some of the many other tools that may be helpful.

One is [libc++ Hardening](https://libcxx.llvm.org/Hardening.html). Real world experience [suggests it can have a large impact](https://security.googleblog.com/2024/11/retrofitting-spatial-safety-to-hundreds.html). A lot of the cases *libc++ Hardening*
will catch will also be caught by *ASan* but some cases for example as access within a `std::vector` capacity but below it's
size. In this same vein is MSVC's [Safe Libraries](https://learn.microsoft.com/en-us/cpp/standard-library/safe-libraries-cpp-standard-library?view=msvc-170).

[clang-tidy](https://clang.llvm.org/extra/clang-tidy/) is a clang-based linter tool. It has a [wide variety of checks](https://clang.llvm.org/extra/clang-tidy/checks/list.html)
Many of them focused on *UB* and *memory safety* type checks but there are also checks that are more style based as well. So
you will need to experiment with the various checks in order to find the right combination that works for your project or team.

There is also [Clang Static Analzyer](https://clang.llvm.org/docs/ClangStaticAnalyzer.html) and MSVC [Code Analysis](https://learn.microsoft.com/en-us/cpp/build/reference/analyze-code-analysis?view=msvc-170).


## Thank You

I would like to thank Patricia Aas, Aaron Ballman and Vlad Serebrennikov for their feedback.
