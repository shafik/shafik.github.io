---
layout: post
categories: [software development]
title: Why Summaries are Important to Pull Request
---

# Why Summaries are Important to Pull Request

Code Reviewers are a precious resource in an open source project, often they have limited bandwidth. A critical goal when
drafting a Pull Request (PR) should be, how does one minimize the amount of time it takes a reviewer to do a review. Especially 
we want to minimize *context switches*, which can have [a large impact on productivity](https://dev.to/reviewpad/context-switching-why-it-hurts-code-reviews-and-what-we-are-doing-about-it-2g0g).
Today I am going to argue that the summary is one of the most important aspects of your pull request to get right. A good summary
should reduce the need for the reviewers to go to their editor to examine the larger surrounding code and minimize the amount 
of context switching a reviewer has to do. 

Another way to think about it, is that you're telling a story to your reviewers. Just like in any conversation, if you leave 
out key parts or details your audience will get lost. Perhaps think of it like a good joke, if you have to explain the joke 
after you tell it, it was not a good joke. You want the PR to land well the first time through. 

Ultimately the more effective you are at explaining your change the better feedback you will receive from your reviewer(s). 

## What Should a Good Summary Include

The most basic framework for writing a summary should be: What, Why and How. What did you change, it does not have to 
exhaustive but should provide enough details so a reviewer(s) knows what to expect. Why did you make the change, understanding 
the motivation for the change can often clarify choices you made that without having a rationale could be surprising. Finally, 
how did you make the changes, did you use a different algorithm or data structure, did you add extra checks or assertions, 
what design choices/tradeoffs did you make etc. 


## It Saves the Reviewer Time

Code reviews are a precious resource, their time is limited and usually the most sought after code reviewers have a lot of 
demands on their time. Since their time is precious everything we can do as a pull request author to save their time means 
more time they will have to review PRs and more time they will have to dedicate to each PR. The summary is where the reviewer 
will start and it is where you can impart the most critical information the reviewer needs. The goal should be to maximize the 
amount of information there so that they have enough context to start the review without leaving the PR itself. We can't 
eliminate the need for the reviewer to switch out of the review, but we can minimize it. 

Often folks who are looking at a problem for a while don't realize that the reviewer does not have the context they have. What 
code looks obvious to you may not be obvious to the code reviewer. A good explanation of the bug and solution can go a long 
way in speeding up the review. 

### What to Include for a Bug Fix PR

- In addition to including a reference to any bug reports (if any), we should also summarize the bug itself. The summary should be put in the context of the code changes.
- We should strive to explain the underling problem at the code level. 
- We should strive to explain how the PR will address the underlying bug.
- If the bug is related to understanding a language standard or some other technical reference we should include relevant links or IDs.

### What to include for a feature PR

- Links to any relevant proposals, RFCs, documentation and discussions of the feature.
- A high level discussion of the feature and how it fits into the existing code.

## Good Summaries as Rubber Ducks

[Rubber ducking](https://en.wikipedia.org/wiki/Rubber_duck_debugging) can be a powerful debugging technique but it also true
that sometimes forcing yourself to explain the problem and the solution makes one see issues that did not come up when
testing. Often in explaining the problem you realize that you can't quite explain it well, maybe you did not 
understand the problem as well as you thought you did. These are all realizations that lead to refactoring or rethinking your
solution. Often you will realize there is a flaw or you need more tests. Often one will catch other problems during a new round of 
testing. This is excellent, you have improved your code and you saved your code reviewer(s) time and hopefully it will reduce 
the number of iterations on the review. 

## Poor Summaries have Outsized Impacts Downstream

In large open source project there will often be many [downstream consumers](https://stackoverflow.com/a/2739476) of your 
project, often with their own downstream local changes. A lot of what we do upstream can have positive or negative impacts on 
folks maintaining forks downstream. Good commit summaries are one of the areas in which we can do some good if we are 
diligent. Often things will break downstream do to conflicts with upstream changes. How quickly a problem can be triaged 
downstream often depends on the quality of summaries. Seeing a long list of bug 
report numbers is not informative at all. While with detailed summaries, one can often search for specific keywords and often 
key in quickly on specific commits likely to be the cause. 

## Even One Liners?

I am going to say, especially the one liners. Small changes often lack the context that gives larger changes more meaning. 
Often developers will take for granted the knowledge they have gained in the time they spent understanding an issue, debugging 
code or just having spent a lot of time supporting that code for a while. The larger interactions are obvious to them and as 
well as what impacts it has to other parts of the code.

