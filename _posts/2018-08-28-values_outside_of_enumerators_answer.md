---
layout: default
title: C++ Poll Answer for August 28th 2018 
---

Apologies C style casts due to twitter character limits. B is undefined behavior.

Part inspired work and by 
<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">I&#39;m pretty sure I have this right, but this is Undefined Behavior, right?<br><br>enum struct Values {<br>  val1 = 1,<br>  val2 = 2<br>};<br><br>Values val = static_cast&lt;Values&gt;(3);</p>&mdash; Jason Turner (@lefticus) <a href="https://twitter.com/lefticus/status/1027928945604218881?ref_src=twsrc%5Etfw">August 10, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Relevant sections of the standard are [\[dcl.enum\]p8](http://eel.is/c++draft/dcl.enum#8):

>For an enumeration whose underlying type is fixed, the values of the enumeration are the values of the underlying type.
Otherwise, the values of the enumeration are the values representable by a hypothetical integer types with minimal range exponent M such that all enumerators can be represented.
The width of the smallest bit-field large enough to hold all the values of the enumeration type is M.
It is possible to define an enumeration that has values not defined by any of its enumerators.
If the enumerator-list is empty, the values of the enumeration are as if the enumeration had a single enumerator with value 0.<sup>101</sup>

and [\[expr.static.cast\]p10](http://eel.is/c++draft/expr.static.cast#10):

>A value of integral or enumeration type can be explicitly converted to a complete enumeration type.
If the enumeration type has a fixed underlying type, the value is first converted to that type by integral conversion, if necessary, and then to the enumeration type.
If the enumeration type does not have a fixed underlying type, the value is unchanged if the original value is within the range of the enumeration values ([dcl.enum]), and otherwise, the behavior is undefined.
A value of floating-point type can also be explicitly converted to an enumeration type.
The resulting value is the same as converting the original value to the underlying type of the enumeration ([conv.fpint]), and subsequently to the enumeration type.

See it [live with Wandbox](https://wandbox.org/permlink/s6judERNBKYIgspG).

For an enum with a fixed underlying type all values of the underlying type are valid.
For an enum without a fixed type, the range is limited and based on the values of enumerators, specified in [\[dcl.enum\]p8](http://eel.is/c++draft/dcl.enum#8). 

roughly limited to the bits needed to represent the values.
