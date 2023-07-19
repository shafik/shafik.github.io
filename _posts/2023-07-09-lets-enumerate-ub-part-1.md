---
layout: post
categories: [C++]
title: Let's enumerate the UB
---

# Let's enumerate the UB

Some time ago I started working on [P1705 Enumerating Core Undefined Behavior](https://wg21.link/p1705r0), I have collected
a large set of *undefined behavior* (*UB*) during that time. There is going to be a lot
of work involved in getting the annex into shape and editing it into the standard. While this work is ongoing,
I will take some time to write blog posts to explore the set of undefined behaviors. In these posts I will cover: why we
have each *undefined behavior*, some consequences of running afoul of each *UB* and tools we can use to catch *UB*. My plan is to enumerate them
in the order they appear in the standard.

## IFNDR (What is that?)

Before we go on, let's talk about a term we find in the standard, *ill-formed no diagnostic required*; also
known as *IFNDR*. This is like *undefined behavior* in that the standard places no requirements on a program that
contains *IFNDR* constructs, see [\[intro.compliance.general\]p2.2](https://eel.is/c++draft/intro#compliance.general-2.2)
which says (*emphasis mine*):

> If a program contains a violation of a rule for which no diagnostic is required, this document *places no requirement on implementations* with respect to that program.

compared to the [defintion of undefined behavior](https://eel.is/c++draft/intro.defs#defns.undefined) which says:

> behavior for which this document imposes no requirements

The main difference is that *IFNDR* is a static property of your code (it is a propety of the program). *IFNDR* means
your program is *ill-formed* but the compiler is not required to diagnose it. Whereas *undefined behavior* is a runtime
property, or a property of the execution of your program. If you run a program that contains *IFNDR* the behavior would be *undefined*.
I bring this up because I will also be covering *IFNDR* cases as
well as *undefined behavior* in the annex and I will also cover these in my blog posts too. For more in depth conversation on
the difference between *IFNDR* and *undefined behavior* we can read
["undefined behaviour" vs "ill-formed & no diagnostic required"](https://groups.google.com/a/isocpp.org/g/std-discussion/c/lk1qAvCiviY).

## Undefined Behavior in the Lexer

The first section of the standard that we run into undefined behavior is the *Lexer*, which also includes one case of *IFNDR*.
Fortunately the two cases of *undefined behavior*, if all goes well, will be removed in *C++26*. Additionally many implementation either
support them with well-formed behavior or treats them as ill-formed. So we will discuss the *UB* but we don't need to discuss
any consequences of invoking them, nor do we need to discuss tools to catch them.

### A Line Splice that Results in UCN

The first lexer *undefined behavior* can be found in [\[lex.phases\]\p2](https://eel.is/c++draft/lex.phases#1.2) which says:

> ... if a splice results in a character sequence that matches the syntax of a universal-character-name, the behavior is undefined

an example of this would be:

```cpp
const char* p = "\\
u0041";
```

In this case many implementations already support this as well formed [(see it live)](https://godbolt.org/z/ojjEE9zxP) and the [P2621: UB? In My Lexer?](https://wg21.link/p2621) proposal should make this *well-formed* in *C++26*.

### Unterminated ' or "

Our next *undefined behavior* can be found in [\[lex.pptoken\]p2](https://eel.is/c++draft/lex.pptoken#2),though we have to
look at a lot of text for this one. Here is the important part:

> A preprocessing token is the minimal lexical element of the language in translation phases 3 through 6. In this document, glyphs are used to identify elements of the basic character set ([lex.charset]). The categories of preprocessing token are: header names, placeholder tokens produced by preprocessing import and module directives (import-keyword, module-keyword, and export-keyword), identifiers, preprocessing numbers, character literals (including user-defined character literals), string literals (including user-defined string literals), preprocessing operators and punctuators, and single non-whitespace characters that do not lexically match the other preprocessing token categories. If a U+0027 APOSTROPHE or a U+0022 QUOTATION MARK character *matches the last category, the behavior is undefined.*

Basically, if we have a proprocessing token that is an unbalanced `'` or `"` then we have *undefined behavior* for example:

```cpp
#define STR_START "  // Undefined behavior, pre-processor token matching apostrophe
#define STR_END "    // Undefined behavior, pre-processor token matching apostrophe

int puts(const char*);

int main() {
  puts(STR_START hello world STR_END);
}
```

In this case many implementations diagnose this as *ill-formed* [(see it live)](https://godbolt.org/z/KaMhhW3cv). I am
guessing way back in time before C was standardized and the *C preprocessor* was a seperate step, something like this may
have just worked[^1]. The [UB? In My Lexer?](https://wg21.link/p2621) proposal should make this *ill-formed* in *C++26*.

## Reserved identifiers

Lastly, we have the *IFNDR* case specified in [\[lex.name\]p3](https://eel.is/c++draft/lex.name#3) which tell us which *identifiers* are reserved for use of the implementation and thus not available for use by user code:

> In addition, some identifiers appearing as a token or preprocessing-token are reserved for use by C++ implementations and shall not be used otherwise; no diagnostic is required.
> - Each identifier that contains a double underscore __ or begins with an underscore followed by an uppercase letter is reserved to the implementation for any use.
> - Each identifier that begins with an underscore is reserved to the implementation for use as a name in the global namespace.

Some examples that cover these cases are ([see it live](https://godbolt.org/z/Mooj4fcvq)):

```cpp
int _z; // No diagnostic required, _z is reserved because it starts with _ at global scope

int main() {
    int __x;  // No diagnostic required, __x is reserved because it starts with __
    int _Y;   // No diagnostic required, _Y is reserved because it starts with _
              // followed by a capital letter
    int x__y; // No diagnostic required, x__y is reserved because it contains __
}
```

While this is *IFNDR* we can see that clang provides the `-Wreserved-identifier` flag to diagnose this. C also has similar rules but [not identical rules](https://en.cppreference.com/w/c/language/identifier#Reserved_identifiers) for reserved identifiers. Some of main differences include `__` only being reserved in the beginning of *identifiers* and the concept of *potentially reserved identifiers* which depends on the implementation and are not portably usable. Some *potentially reserved identifiers* in C are function names beginning with `is` or `to` followed by a lowercase letter. The [C99 rationale document](https://www.open-std.org/jtc1/sc22/wg14/www/C99RationaleV5.10.pdf) explains the need for *reserved identifiers* as follows:

> Also reserved for the implementor are all external identifiers beginning with an underscore, and
all other identifiers beginning with an underscore followed by a capital letter or an underscore.
This gives a name space for writing the numerous behind-the-scenes non-external macros and
functions a library needs to do its job properly.

For C++ we can also consider the difficulty of identifying consistently when we are processing *STL* code versuses user code.

We can see that [clang builtin functions](https://clang.llvm.org/docs/LanguageExtensions.html#builtin-functions) and [gcc builtin functions](https://gcc.gnu.org/onlinedocs/gcc/Other-Builtins.html) start with `__`. For example,  [\_\_builtin_unreachable](https://clang.llvm.org/docs/LanguageExtensions.html#builtin-unreachable) which is used to mark a location that should never be reached.

Thank you, to those who provided feedback on this write-up: Aaron Ballman, Corentin Jabot and Erich Keane,

Of course in the end, all errors are the author's.

## Further References

We can see that the [UB? In My Lexer?](https://wg21.link/p2621) proposal the was not the first attempt at removing these *undefined behaviors* from lexing. The was [DR 787](https://www.open-std.org/jtc1/sc22/wg21/docs/cwg_defects.html#787) and [N3881: Fixing the specification of universal-character-names](https://open-std.org/jtc1/sc22/wg21/docs/papers/2014/n3881.pdf) both attempted to wittle down *UB* in the lexer.

We can also read [The Development of the C Language](https://www.bell-labs.com/usr/dmr/www/chist.html) which talks about the C
proprocessor and it lack of specifications before standardization and how some features that we used today worked were "previously available only by accidents of implementation".

[^1]: [Traditional lexical analysis](https://gcc.gnu.org/onlinedocs/cpp/Traditional-lexical-analysis.html#Traditional-lexical-analysis)
