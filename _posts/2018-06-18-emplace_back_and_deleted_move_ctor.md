---
layout: post 
categories: [polls, undefined behavior, C++, "std::vector"]
title: C++ Poll for June 18th 2018, std::vector and the Case of the Deleted Move Constructor
---

<blockquote class="twitter-tweet" data-partner="tweetdeck"><p lang="en" dir="ltr">In C++11 given<br><br>struct A {<br>  A(int a);<br>  A(A&amp;&amp;)=delete; <br>};<br><br>The following<br><br>  std::vector&lt;A&gt; v;<br>  v.emplace_back(1); <br><br>A. Well-formed<br>B. Requires diagnostic<br>C. Invokes undefined behavior<br><br>Answer first, read answer below.<a href="https://twitter.com/hashtag/Cplusplus?src=hash&amp;ref_src=twsrc%5Etfw">#Cplusplus</a> <a href="https://twitter.com/hashtag/Programming?src=hash&amp;ref_src=twsrc%5Etfw">#Programming</a> <a href="https://twitter.com/hashtag/CppPolls?src=hash&amp;ref_src=twsrc%5Etfw">#CppPolls</a></p>&mdash; Shafik Yaghmour (@shafikyaghmour) <a href="https://twitter.com/shafikyaghmour/status/1008936483124727808?ref_src=twsrc%5Etfw">June 19, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


<BR>
<input type="button" onclick="location.href='{% link _posts/2018-06-18-emplace_back_and_deleted_move_ctor_answer.md %}'" value="goto Answer;"/>
<BR>
