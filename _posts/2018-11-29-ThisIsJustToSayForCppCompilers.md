---
layout: post 
categories: [poetry, undefined behavior, C++, compilers]
title: C++ Poll for November 29th 2018, This is Just to Say for C++ Compilers
---

This Is Just to Say

I have translated<br>
the program<br>
that was in<br> 
the translation units<br>

and which<br>
you were probably<br>
expecting<br>
predictable results<br>


Forgive me<br>
they required no diagnostic<br>
so undefined<br>
and so erroneous<br>

---

Inspired by [This Is Just to Say](https://en.wikipedia.org/wiki/This_Is_Just_to_Say)

--

I originally wrote this around April 2nd 2018 and then [tweeted it](https://twitter.com/shafikyaghmour/status/980898697578921984).

As [I noted in the tweet](https://twitter.com/shafikyaghmour/status/981198025488912384) almost all the replacement words came from the [defintion of undefined behavior in the C++ standard](http://eel.is/c++draft/intro.defs#defns.undefined):

>**undefined behavior**  
behavior for which this document imposes no requirements  
[ Note: Undefined behavior may be expected when this document omits any explicit definition of behavior or when a program uses an erroneous construct or erroneous data.
Permissible undefined behavior ranges from ignoring the situation completely with unpredictable results, to behaving during translation or program execution in a documented manner characteristic of the environment (with or without the issuance of a diagnostic message), to terminating a translation or execution (with the issuance of a diagnostic message).
Many erroneous program constructs do not engender undefined behavior; they are required to be diagnosed.
Evaluation of a constant expression never exhibits behavior explicitly specified as undefined in [intro] through [cpp] of this document ([expr.const]).
— end note
 ]
