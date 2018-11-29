---
layout: default
title: C++ Poll Answer for November 14th 2018 
---

The output will be:

    1111

This is due to [\[alg.min.max\]p3](http://eel.is/c++draft/alg.min.max#3) which says:

>Remarks: Returns the first argument when the arguments are equivalent.
An invocation may explicitly specify an argument for the template parameter T of the overloads in namespace std. 

and [\[alg.min.max\]p11](http://eel.is/c++draft/alg.min.max#11) which says:

>Remarks: Returns the first argument when the arguments are equivalent.
An invocation may explicitly specify an argument for the template parameter T of the overloads in namespace std.

So both `std::max` and `std::min` return the first argument when they are equivilent. Sean Parent 
discusses this issue in his [CppNow 2016 Keynote: Better Code](https://youtu.be/giNtMitSdfQ?t=1429) where he considers this a 
defect.

`std::minxmax()` does better here and if they are equivalent min is the first and max is the second, see [\[alg.min.max\]p19](http://eel.is/c++draft/alg.min.max#19). 

This was also added to [CppQuiz as a question](http://cppquiz.org/quiz/question/248).
