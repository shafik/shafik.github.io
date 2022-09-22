---
layout: post
categories: [C++]
title: C++ initialization, arrays and lambdas oh my!
---

I recently ran a [C++ weekly quiz #198](https://twitter.com/shafikyaghmour/status/1568451213984944128?s=20&t=YhmI1ABnV9L8edhtYfHwfw) with the following code:

```cpp
int main() {
    int arr[5]{1,2};

    return arr[ [](){return 4;}() ]; // What does main return?
}
```

Likely folks will be surprised that this does not compile ([see it live](https://godbolt.org/z/95j8EW7dq)). The standard says
that two consecutive `[` shall only be used to introduce an attribute, see [\[dcl.attr.grammar\]p7](https://eel.is/c++draft/dcl.attr.grammar#7):

> Two consecutive left square bracket tokens shall appear only when introducing an attribute-specifier or within the balanced-token-seq of an attribute-argument-clause.

Originally when attributes was introduced into C++ this restriction did not exist but later on it was realized that `[[` could
be ambiguous in some cases. Such as introducing lambda inside an array subscript. See [defect report 968](https://cplusplus.github.io/CWG/issues/968.html):

> The [[ ... ]] notation for attributes was thought to be completely unambiguous. However, it turns out that 
two [ characters can be adjacent and not be an attribute-introducer: the first could be the beginning of an
array bound or subscript operator and the second could be the beginning of a lambda-introducer. This needs to
be explored and addressed.

The reason this matters is that allowing this would complicate the compiler. It would require unlimited look ahead in order
to disambiguate. Introducing this complexity to solve the few cases where it mattered are not compelling enough to compensate for
the additional complexity. In order to disambiguate you can add parenthesizes. For example:

```cpp
arr[[someSetOfCharacter](){return 4;}()]
  // ^ unlimited lookahead needed to decide if this is a capture of attribute-list 
```

This in some ways is similar to maximal munch type problems. Which I have [written about previously](https://shafik.github.io/c++/maximal%21munch/2020/12/28/maximal_munch_and_cpp.html).
The most infamous pre C++11 case of maximal munch was that of closing template parameter lists:

```cpp
std::vector<std::vector<int>> v;
                          //^^ Prior to C++11 we required a space between each >
```

This was seen as a significant gothca so an exception was added to fix it. 

Although unlike maximal munch `[[` is not a token unlike other cases such as `>>`, `/*`, `*=` etc which are all 
[operators or punctuators](https://eel.is/c++draft/lex.operators#nt:operator-or-punctuator). Also unlike classical
maximal munch problems adding a whitespace between each `[` does not disambiguate.
