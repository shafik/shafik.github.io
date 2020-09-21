---
layout: post
categories: [C++, learning]
title: CppCon 2020 Trip Report
---

CppCon 2020 was online this year due to Covid-19. I was not sure what to expect from an online only conference. I had
heard mildly positive feedback from recent online only conferences but this would be my first experience. For the most part
the experience exceeded my expectations. There were were technical problems here and there but mostly
it ran smoothly. It was fatiguing to watching videos for so long but since I was home I was able to stretch in
between sessions and move around and that usually helped.

They used Remo conference software to host the conference. While it was definitely not the same as being in person the
experience was pretty good. They had several virtual rooms at the top of the interface in which the talks took place.
Each room had virtual tables where you could "sit" and if you want to turn on your microphone and camera could socialize
with the folks at your table by voice and or video. I did not spend much time socializing
this way but the experience when I did was nice. Remo also had a chat which was split into a common
chat area and a room chat area where you could chat with those at your table. While a talk was going on you could chat
with the whole room. It was great for the audience since discussions during the presentation often turned technical and
a lot of basic and not so basic questions could be answered by the other attendees pretty quickly. Each room also had a
Q&A board where the audience could post questions for the speaker and vote on the questions. Sometimes the speaker
choose to answer during the talk and some after but besides a few glitches I saw it worked well. Between talks there
was a Remo "Hallway Track" where you could grab a table and socialize or just chill out.

One takeaway from the remote conference experience is that having the group chat during the talk was helpful. Folks were
friendly and helpful to those asking question during the talk got helpful answers pretty quickly from the rest of the
audience. During lighting talks this also served to show general interest in the topic and could be used to gauge whether
allowing the speaker to run over was ok. I personally felt bad not being very social, this was partly because being home
meant I had other things to do but also it just lacks the spontaneous experience of just seeing someone and walking up and
saying hello. The whole clicking on a table and starting a conversation felt a bit awkward.

What about the talks? There were a lot of great talks and it is hard to pick but I am going to try and list my five
must see talks and why, I also include a link to my live tweets on each talk:

- [Dynamic Polymorphism with Metaclasses and Code Injection](https://twitter.com/shafikyaghmour/status/1306256011028643841)

  [@TartanLlama](https://twitter.com/TartanLlama) talked about static reflection and code injection and centered the talk on
  how one could replace dynamic polymorphism using these features.
  This relies on the [static reflection work](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1240r1.pdf) which is
  targeted for C++23 and code injection (or [meta-classes](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0707r4.pdf))
  which does not currently have a target but is being worked on.
  It felt like an impressive amount of content from laying out the use case and then stepping through each piece of the
  replacement which was all pretty new stuff.
  On top of that the features feel pretty powerful and after watching the talk you are itching to get your hands on them
  because they just look awesome.

- [Just-in-time compilation](https://twitter.com/shafikyaghmour/status/1305593864598634496)

   [@jfbastien](https://twitter.com/jfbastien) gave an overview of the field of Just-in-time compilation as seen through the
   view of several key papers in the field. He covered a lot of different papers each with their own interesting insights
   combined with JF's experience in the field made for a wonderful talk.

- [Just-in-Time The Next Big Thing?](https://twitter.com/shafikyaghmour/status/1306296651091292160)

  [@ben_deane](https://twitter.com/ben_deane) and [@krisjusiak](https://twitter.com/krisjusiak) started the talk with a quick
  introduction to current JIT implementations and then moved into what is proposed by
  [P1609R1 "C++ Should Support Just-in-Time Compilation"](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1609r1.html).
  The talk quickly moved into some practical examples, they were pretty impressive and only got more impressive as the talk
  went on. To be honest about half-way through I stopped following all the details because it was a lot of really detailed
  examples. Regardless it was a great talk and I will just have to go back and study the details of the later examples to
  fully get them.

- [Structure and Interpretation of Computer Programs: SICP](https://twitter.com/shafikyaghmour/status/1306973453262569474)

  [@code_report](https://twitter.com/code_report) talked about the [classic computer science book SICP](https://mitpress.mit.edu/sites/default/files/sicp/full-text/book/book.html)
  and why it matters. Some quotes:

  >"focused attention on the central idea of abstraction ... the idea of function as data ... [and] three different
programming paradigms: functional, object oriented and declarative"

  and

  > "A programmer should acquire good algorithms and idioms".

  Next he explored the ideas "It's just data", "Data Abstraction" and "OOP" as they related to SICP.
  Then he dived into several problems from SICP and then solved them in C++ using C++20 and sometimes C++23
  and beyond solutions focusing on the principles that were discussed earlier. They were well chosen problems and the
  solutions were indeed beautiful. The masterful use of modern C++ is super nice and makes me wish I could solve all
  my problems so elegantly.

- [What’s in a Name? What’s a Name In?](https://twitter.com/shafikyaghmour/status/1306611476161994752)

  Walter talks was about names, which seems like a simple enough topic. As we get into the talk bit by bit we realize that it
  is deceptively simple and a lot of fine details lie underneath. He walks us through the standard to see exactly what it means.
  We go from *identifiers*, *entities*, *names*,  *reserved names* and *declarations*. Then we start talking about context and regions of
  code, because *"names don't exist in vacuo"*. He walks us through name lookup and the details (not all b/c that would take too long)
  but enough to cover some tricky and interesting cases. I am happy because I came away with a few good ideas for future (maybe present)
  [quizzes](https://twitter.com/search?q=%23CppPolls&f=live).


Thank you Patricia Aas for the feedback on this write-up.

Of course in the end, all errors are the author’s.
