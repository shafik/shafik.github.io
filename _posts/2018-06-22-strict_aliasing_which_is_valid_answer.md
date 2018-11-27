---
layout: default
title: C++ Poll Answer for June 22th 2018 
---

A. is a valid aliasing. This is covered in [\[basic.lval\]p11](http://eel.is/c++draft/basic.lval#11):

>If a program attempts to access the stored value of an object through a glvalue of other than one of the following types the behavior is undefined:<sup>55</sup>
>
>- (11.1) the dynamic type of the object,
>- (11.2) a cv-qualified version of the dynamic type of the object,
>- (11.3) a type similar (as defined in [conv.qual]) to the dynamic type of the object,
>- (11.4) a type that is the signed or unsigned type corresponding to the dynamic type of the object,
>- (11.5) a type that is the signed or unsigned type corresponding to a cv-qualified version of the dynamic type of the object,
>- (11.6) an aggregate or union type that includes one of the aforementioned types among its elements or non-static data members (including, recursively, an element or non-static data member of a subaggregate or contained union),
>- (11.7) a type that is a (possibly cv-qualified) base class type of the dynamic type of the object,
>- (11.8) a char, unsigned char, or std::byte type.

[Footnote 55](http://eel.is/c++draft/basic.lval#footnote-55) says:

>The intent of this list is to specify those circumstances in which an object may or may not be aliased.

A. is allow by [\[basic.lval\]p11.4](http://eel.is/c++draft/basic.lval#11.4)

B. is not allowed by any of the bullets

C. might be surprise and diff than C, only char, unsigned char are valid for aliasing see [\[/basic.lval\]p11.8](http://eel.is/c++draft/basic.lval#11.8)

Also worth checking out my Stack Overflow [answer to "What is the strict aliasing rule?"](https://stackoverflow.com/a/51228315/1708801)

Some replies to this poll that are worth pointing out:

<blockquote class="twitter-tweet" data-partner="tweetdeck"><p lang="en" dir="ltr">Extra credit question, only for the truly motivated student: does C actually define &quot;character type&quot; anywhere, and if so, where?</p>&mdash; Steve Canon (@stephentyrone) <a href="https://twitter.com/stephentyrone/status/1010209411405615107?ref_src=twsrc%5Etfw">June 22, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

and

<blockquote class="twitter-tweet" data-partner="tweetdeck"><p lang="en" dir="ltr">I liked your use of F for an option lettering ;-)<br><br>We should have a simple way to treating on object like a bag of bits, I think bit_cast gets us partly there.<br><br>After seeing how pre ANSI C typed punned <a href="https://t.co/aAwP7Yp2UF">https://t.co/aAwP7Yp2UF</a> it does have appeal in its simplicity.</p>&mdash; Shafik Yaghmour (@shafikyaghmour) <a href="https://twitter.com/shafikyaghmour/status/1010190915648933888?ref_src=twsrc%5Etfw">June 22, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

