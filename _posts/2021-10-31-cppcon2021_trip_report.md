---
layout: post
categories: [C++, learning]
title: CppCon 2021 Trip Report
---

CppCon 2021 was a hybrid format this year due to Covid-19 still not being done with us. Folks were allowed on site with 
proof of vaccination but folks were also allowed to attend completely remote. Last year was fully online and that
was a [pretty positive experience for me](https://shafik.github.io/c++/learning/2020/09/21/cppcon2020_trip_report.html). This 
year I still wasn't sure what to expect since I was attending remotely because it
was a hybrid format. The schedule was shifted earlier US time-wise to allow those in Europe to still participate 
live. It also meant that some content would not be broadcast the same day but the next day and as I learned later
some content was only at the venue. Like last year, it was fatiguing to watch videos for long stretches of time but getting up 
to moving around and stretching helped. 

There were a lot of changes from last year for the remote users. There were multiple interfaces to the conference for one 
thing. I felt like they outdid themselves this year, it felt pretty fancy and slick but also slightly overwhelming at first. 
One interface was a *Online Platform/Web Interface* through [Digital Medium](https://events.digital-medium.co.uk) where you get quick 
access to 
the talks from each track, the schedule, recordings of previous talks, quick links to resources etc. I found myself mainly 
using this, it was quick to get to the talk you wanted to and to find previous recorded talk through that interface.
There was a *Virtual Venue* though [Gather Town](https://www.gather.town/) which featured an accurate overhead 
representation of the *Gaylord Rockies*
conference venue. You had an avatar, you could walk through the virtual conference site and goto the various
talks that way as well. The Gather Town interface allowed you to enable voice and video, if you were next to other virtual
participants you could see and hear them if they had their voice or video on. The other piece of the conference was a CppCon
Discord server. The conference used this to communicate updates, makes announcements and a help-desk was also available.
You could also use Discord to socialize with conference participants on various channels.

The virtual venue felt like playing an overhead role playing game. They added a set of puzzles that could be found around
the virtual venue if you looked hard enough. Solving each puzzle earned you points and you could check your progress on a 
leader board in the virtual environment. The puzzles mainly consisted of C++ trivia of varying degrees of difficultly. Several
of the puzzles were more difficult pieces of C++ history. I learned something I did not know about before that in 1994 
Erwin Unruh created a template meta program that [calculated the prime numbers in error messages](https://rtraba.files.wordpress.com/2015/05/cppturing.pdf) 
showing that templates are Turing complete. You can still find the [original version](http://www.erwin-unruh.de/primorig.html) 
but it does not work on modern compilers.  

I found the mixed interfaces to be stumbling block, it took half a day to really figure out what worked for me and how to find
what I wanted. They provided a pretty detailed document with nicely marked up pictures but it was a lot to absorb, it 
didn't really click until you actually had to use it. During the weekend before, I found the virtual venue fun and solved a 
lot of puzzles but once the talks started I found that being able to quickly get to a talk more important and 
dropped the virtual venue. That also meant that I did not socialize at all during the event. I am not sure how I would have 
changed 
things. I think each individual interface worked very well but once I started out using the web interface it was hard to go 
back to the virtual one. Part of that was the reality of still being in my day to day world which has its own demands while 
attending the conference remotely. 


What about the talks? Last year I did [the five must see talks](https://shafik.github.io/c++/learning/2020/09/21/cppcon2020_trip_report.html).
This year I want to highlight the five talks I felt either:

- Presented a complicated subject in clear and concise way
- Gave us a vision for C++ in an inspiring way
- Helped us to understand what on the outset does not seem like a deep or widely applicable problem and showed how, yes, you should care about this and how it applies more widely. 
 
I also picked one of the lighting talk sessions. I will link to each speakers twitter account if it exists as well as the talk once it becomes available online. Here are my picks for talks to watch from the CppCon 2021:

- Embracing User defined literals for safely for types that behave as though builtins

  By [@PabloGHalpern](https://twitter.com/PabloGHalpern) 
  
  This talk not only explained the mysterious subject of
  user defined literals but also delved into the safety aspects around this language feature. This was the first good 
  explanation I have seen of this feature. I did not realize how rich a feature this was and the problems that it allows
  one to solve. There are also plenty of pitfalls that you have to watch out for but after watching this talk I think you 
  will be in a good place to use them wisely. 

- C++20 Templates: The next level: Concepts and more 
  
  By [@Andreas__Fertig](https://twitter.com/Andreas__Fertig) 
  
  I have to say, this was my favorite talk, it was a well polished talk. I have spent some time looking at concepts 
  but never really in depth, I guess because I am not  
  writing library like code most of the time. I came away with a much better understanding of concepts and 
  the rationale for a lot of tricky pieces such as *requires requires* and *compound requirements*. He managed to fit a lot
  of information in each slide in a way that did not feel overwhelming. I know I will be able to go back to the slides and
  they will be a great references.
  
- Lighting talks with Michael Caisse

  Lighting talks are hit or miss but the first set of lighting talks hosted by [Michael Caisse](https://twitter.com/MichaelCaisse) 
  were pretty much all solid and well worth watching, each talk although short felt liked it was packed solid
  content. From Walter Brown's talking about "Loop Unrolling" to [@ben_deane](https://twitter.com/ben_deane) talking about 
  "The Process is the Problem". They each landed well in the short time they had. Don't miss these lighting talks.

- Extending and Simplifying C++: Thoughts on pattern Matching using `is` and `as`

  By [@herbsutter](https://twitter.com/herbsutter) and [@seanbax](https://twitter.com/seanbax)
  
  Herb's talk was a visionary talk about how to make C++ more expressive using `operator is` and `operator as`. He 
  explained how we can use these operators to replace a whole set of type query and type cast operations. He also demonstrated
  how they could also be applied to standard library types such as `std::variant` and `std::optional`. He was joined on stage
  for a set of three demos by Sean who has implemented these features in his [Circle C++ Compiler](https://www.circle-lang.org/).
  I felt like the talk made an solid argument for how these features could transform the language to make it more expressive 
  and safer. They provided [godbolt links on the slides](https://twitter.com/shafikyaghmour/status/1454581583705358337) so 
  folks can try this out for themselves. Hopefully folks will take the opportunity to write some code using this feature 
  and help drive constructive feedback on the proposal. 

- Correctly Calculating min, max, and More: What Can Go Wrong?

  By Walter Brown
  
  Many in the C++ community [have known that std::min and std::max have design issues](https://twitter.com/shafikyaghmour/status/1062869521084469248?s=20).
  So at first I was like, "I know what this is about, maybe I should skip it". I am happy I did not listen to that part
  of me. He took a wider perspective and explained what the issue was and why this is not specific to `std::min` or `std::max`
  but is a wider more applicable issue. He explained why this is related to `operator <` and introduced a `out_of_order` and
  and `in_order` and why looking at the various problem through a different lens allows us to avoid algorithmic pitfalls. He
  walks through the many different "spellings" or `operator <` and various implementations of each one. There is a lot of food 
  for thought packed into a relatively short time. You will be thinking about this talk longer afterwards.

- What You Can Learn from Being Too Cute: Why You Should Write Code That You Should Never Write
  
  By [@Daisy Hollman](https://twitter.com/The_Whole_Daisy)
  
  I loved the energy of this talk, she spent time to learn some really interesting dark corners of C++ and figured out how
  to do interesting and cursed things with them. All the while having fun and I would say learning is always profit. 
  Then she distills the essence of each piece and walks us step by step through
  it so that we can understand why it works, what part of the language explains this and the caveats around it. Learning 
  about dark corners of C++ can be rewarding but figuring out how to communicate that to broader community is way more 
  rewarding and also gives back and helps others learn. Some will know this to be an area dear to me as I often plumb the dark
  corners of C++ writing my [Weekly C++ quizzes on twitter](https://twitter.com/search?q=%23CppPolls&src=typed_query&f=live). 
  I also often post [cursed code](https://twitter.com/search?q=%22cursed%20code%22%20%40shafikyaghmour&src=typed_query&f=live)
  on twitter but usually leave it as an exercise to the user to figure out what is going on. Maybe this will inspire me to
  do some more detailed posts in the future 🤔

I learned a lot during this conference and the only downside was that Walter Brown's "Computing in the 1960's" talk was [Open 
Content talk](https://cppcon.org/open-content-submissions/) and it won't be posted online. The past few years I have grown 
more interested in the history of how we got
to where we are with C++ and programming languages in general. A few years ago [I spent time rereading "Design and Evolution of C++](https://twitter.com/shafikyaghmour/status/1212238089168416768?s=20). 
It gave me perspective on how a programming language evolves and can make dramatic improvement over time with thought and 
carefully balancing of trade-offs. I am sure Walter's talk was full of perspective and hopefully he will give it again.
