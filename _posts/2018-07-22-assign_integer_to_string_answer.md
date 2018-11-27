---
layout: default
title: C++ Poll Answer for July 22th 2018 
---

This program is well-formed. We can see from [cppreference](https://en.cppreference.com/w/cpp/string/basic_string/operator%3D) we have

    basic_string& operator=( CharT ch );

and int is converted to char. 

This was subject of a [LWG defect report 2372: Assignment from int to std::string](http://cplusplus.github.io/LWG/lwg-closed.html#2372) which was closed NAD and which says:

> The following code works in C++:
>
    int i = 300;
    std::string threeHundred;
    threeHundred = i;
>
> "Works" == "Compiles and doesn't have an undefined behavior". But it may not be obvious and in fact misleading what it does. This assignment converts an int to char and then uses string's assignment from char. While the assignment from char can be considered a feature, being able to assign from an int looks like a safety gap. Someone may believe C++ works like "dynamically typed" languages and expect a lexical conversion to take place.  

There where several good tweets in reply to this poll:

<blockquote class="twitter-tweet" data-partner="tweetdeck"><p lang="en" dir="ltr">There isn&#39;t, it&#39;s just the assignment operator that&#39;s broken. Easily fixable too:<br><br>template &lt;Same&lt;CharT&gt; T&gt;<br>basic_string&amp; operator=(T const);</p>&mdash; Christopher Di Bella (@cjdb_ns) <a href="https://twitter.com/cjdb_ns/status/1021447789245620225?ref_src=twsrc%5Etfw">July 23, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

and

<blockquote class="twitter-tweet" data-partner="tweetdeck"><p lang="en" dir="ltr">I also think `+= CharT` seems like a bad idea</p>&mdash; undefined behavior is pretty cool (@ubsanitizer) <a href="https://twitter.com/ubsanitizer/status/1021487153157693440?ref_src=twsrc%5Etfw">July 23, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

