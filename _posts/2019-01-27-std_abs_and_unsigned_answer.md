---
layout: default
categories: [poll answers] 
title: C++ Poll Answer for January 27th 2019 
---

`foo` is well-formed and `bar` is not, this is covered in [\[c.math.abs\]p3](http://eel.is/c++draft/c.math#abs-3):

>Remarks: If abs() is called with an argument of type X for which is\_­unsigned\_­v<X> is true and if X cannot be converted to int by integral promotion, the program is ill-formed.
[ Note: Arguments that can be promoted to int are permitted for compatibility with C.— end note] 

This came about due to [LWG defect 2192](http://www.open-std.org/jtc1/sc22/wg21/docs/lwg-defects.html#2192). 

Interesting to note that [gcc and clang disagreed on this one for a while](https://stackoverflow.com/q/29750946/1708801).
