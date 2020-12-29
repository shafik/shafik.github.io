---
layout: post
categories: [C++,maximal munch]
title: Maximal Munch and C++
---

In [C++ Weekly Quiz #106](https://twitter.com/shafikyaghmour/status/1317270944780484609) we had the following set of declarations:

```
template <int>
using A = int;
int x,y,*z;
template<class T> class S;
class C;
```

and we were asked given these further declaration or expressions which of the following where well-formed:

```
A. void f(A<0>=0);
B. x=y/*z;
C. S<::C>* cls;
D. x+++++y;
```

What all of these had in common is that they were ill-formed or once were ill-formed due to *maximal munch*.

What is *maximal munch*? 

*Maximal munch* is a principle used during lexical analysis that says that and I will quote [\[lex.pptoken\]p3.3](http://eel.is/c++draft/lex#pptoken-3.3):

>the next preprocessing token is the longest sequence of characters that could constitute a preprocessing token

This simplifies lexing and usually produces the "right" results, although we occasionally end up with cases that can be puzzling to those who are not aware and have not seen this before.

For example, let take `B` from above:

```
int x;
int y;
int *z;
//... some other work
x=y/*z;
// ^ Is this the start of a comment or division followed by a pointer dereference?
```

Is the `/*` the start of a comment or is it a division followed by a pointer dereference? *Maximal munch* says we have to take the longest sequence of characters that forms a token and this will disambiguate to the start of a comment. We fix this by adding parentheses:

```
x=y/(*z);
```

Prior to C++11 the most infamous case of *maximal munch* had to do with consecutive `>` when closing template parameter lists:

```
 std::vector<std::vector<int>> v;
                          //^^ Prior to C++11 we required a space between each >
```

Although a relatively minor problem in the big scheme of things, it was annoying to have to remember to always leave a space when closing template parameter lists. In general this was considered an embarrassing situation since it was hard to understand why this could not be made to work.

This was fixed in C++11 by the proposal [N1757: "Right Angle Brackets"](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2005/n1757.html) which added a special rule for this case, see [\[temp.names\]p3](http://eel.is/c++draft/temp.names#3):

>Similarly, the first non-nested >> is treated as two consecutive but distinct > tokens, the first of which is taken as the end of the template-argument-list and completes the template-id.

In general when we want to solve a lexing issue that comes up due to maximal munch we need to add an exception to the language to carve out that special case.

Let's look at another example, this time one of the more infamous [Stack Overflow question](https://stackoverflow.com/a/24947922), why does this not work?:

```
x+++++y;
```

If we follow the maximal munch rule then we end up with the following:

```
x ++ ++ + y ; // Ill-formed trying to apply postfix increment to x twice
              // Assuming primitive types like int
```

We have two *postfix increments* of `x` followed by an addition of result and `y`. The problem is that *postfix increment* requires a *modifiable lvalue* but results in a *prvalue* therefore the second *postfix increment* is ill-formed. Folks often expect that it will be lexed like this:

```
x ++ + ++ y ; // Well-formed addition of the result of postfix increment of x and 
              // prefix increment of y
```

If we were to dump *maximal munch* what heuristic would we use to determine which of the possible token is correct?

```
x + + + + + y; // Additions and four unary pluses?
x + ++ ++ y;   // Addition and two prefix increments?
x ++ ++ + y;   // Two postfix increment and an addition?
// It goes on...
```

Would we have to try each possible lex to see which were valid? What if multiple were valid? This would be more expensive and adds complexity and with little real gain.

The other two cases noted above `S<::C>* cls;` and `void f(A<0>=0);` are interesting examples since before C++11 both were ill-formed due to maximal much but the `<::` case was fixed in C++11 via [defect report 1104](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2011/n3237.html#1104) which added the following exception for this case in [\[lex.pptoken\]p3.2](http://eel.is/c++draft/lex.pptoken#3.2):

>Otherwise, if the next three characters are <\:\: and the subsequent character is neither : nor >, the < is treated as a preprocessing token by itself and not as the first character of the alternative token <\:.

Otherwise `<:` will be treated as a [digraph](http://eel.is/c++draft/lex.digraph) which is replaced by `[` so the above example ends up as:

```
S[:C>* cls;
```

which is ill-formed.

There are many other seemingly bizarre cases such as:

```
void f1(A*=0);
       //^^ Is this A* and assignment 0
       //or is this A compound assignment 0 
```

Maximal munch resolves this ambiguity as `A` followed by *compound assignment* and then zero which in this context is ill-formed. Regardless of whether treating this as *A pointer* and a *default value* would be well-formed. Hat tip to [@flacs](https://twitter.com/f1ac5) who [ran into this one the other day](https://twitter.com/f1ac5/status/1313186140845998081).


How about [this fun case](https://twitter.com/shafikyaghmour/status/1162990625160978439) that comes up when lexing a `pp-number`:

```
int x = 0x8000000e-0xf0;
               //^^ Is this the end of hexideciaml-integer-literal followed by
               //   a minus and another hexidecimal literal
               // or is this an ill-formed hexidecimal-floating-literal 
```

For this case it might be worth taking a detour to explain the difference between a *preprocessing token* and a *token*. In [proprocessing phases two to six](http://eel.is/c++draft/lex.phases) the preprocessor is dealing with *preprocessing tokens* which are coarser categories then *tokens*. For example *identifiers* and *keywords* are *identifiers* during phases two to six, *integer literals* and *floating-point literals* are *pp-numbers* and *operators* and *punctuation* are *preprocessing-op-or-punc* during phases two to six.

The grammar for a [pp-number](http://eel.is/c++draft/lex#nt:pp-number):

```
pp-number:
  digit
  . digit
  pp-number digit
  pp-number identifier-nondigit
  pp-number ' digit
  pp-number ' nondigit
  pp-number e sign
  pp-number E sign
  pp-number p sign
  pp-number P sign
  pp-number .
```
means that `0x8000000e-0xf0` will be treated as one `pp-number`:

```
digit
---------------
| |     |  |  |
V V.....V  V  V 
0x8000000e-0xf0
 ^       ^^ ^^
 |       || ||
  --------|--- identifier-nondigit
          |
           sign
```

During [lexical phase 7](http://eel.is/c++draft/lex.phases#1.7) each *preprocessor token* is converted to a [token](http://eel.is/c++draft/lex.token) but `0x8000000e-0xf0` is not a valid [literal](http://eel.is/c++draft/lex.literal.kinds#nt:literal), therefore ill-formed. On the other hand `0x8000000e - 0xf0` would result in three proprocessor tokens:

```
0x8000000e -> pp-number
-          -> preprocessing-op-or-punc
0xf0       -> pp-number
```

Which would result in well-formed code.

I will leave you one more interesting case which comes from [\[expr.prim.id.dtor\]](http://eel.is/c++draft/expr.prim.id.dtor#3) and [I tweeted about recently](https://twitter.com/shafikyaghmour/status/1341828917439549441):

```
using T = int;
0 .T::~T();       // integer-literal followed by class member access
                  // followed by a pseudo-destructor call
0.T::~T();        // error: 0.T is a user-defined-floating-point-literal but is ill-formed
```

We spent some time seeing where maximal munch causes weird issues but the vast majority of the time it does the right thing, for example:

```
++x;    // This is not unary plus applied to x twice
        // This is not addition and then unary plus applied to x
var->x  // This is not var minus greater than x 
        // var - > x
int integerA; // This is not 
              // int int tegerA;
```

I hope you enjoyed this, this is very much trivia about how lexing works in practice maximal munch problems while they may be puzzling at first are solved by adding whitespace to disambiguate. If you find more interesting maximal munch examples please send them my way and I will happily give you a hat tip when I use them.

Thank you to those who provided feedback on or reviewed this write-up: Aaron Ballman, JeanHeyd Meneide and Richard Smith.

Of course in the end, all errors are the author's.
