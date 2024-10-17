---
layout: post
categories: [C++,LLVM]
title: Triaging clang C++ frontend bugs 
---

# Triaging clang C++ frontend bugs

I have been triaging clang C++ frontend bugs for about a couple of years now and I wanted to share some of the lessons
I have learnt in the hopes that others in the C++/LLVM community can feel less intimidated and get in on the fun as well.
I have tried to develop some good practices for triaging and I have attempted to distill those here as well.

The clang frontend usually gets several new bugs a day and it is important to our users to at least do and initial assessment
of each bug in a timely manner. Bugs are going to fall into one of many categories (not exhaustive list):

- It is not a bug, this is rare but it does happen and often we need to walk the user through why that is the case and possibly any workarounds to address their issue.
- Undefined behavior, this is kind of category of "not a bug" but sometimes the compilers behavior around UB can be unintuitive or downright not desirable. Often there are good workarounds but sometimes not.
- Regressions, we have regular release of clang and sometimes we break things that worked in the previous release. These are important to flag early and fix.
- Crashes, unfortunately the complier can crash usually giving a dreaded backtrace but sadly sometimes not.
- Different compilers obtain different behavior. These bugs vary a lot. Sometimes it is just a bug in clang or another implementation. Sometimes the C++ standard is not clear or it has a wording bug.
- We have a diagnostic that has a false positive or false negative.
- We don't support an extension the same way another compiler does. We support many GNU and MSVC extensions and sometimes our behaviors don't line up.

This is a long list and it I will acknowledge that not all bugs types require the same amount of work to triage.

If you are interested in helping out and have questions, the [clang channel on the llvm Discord](https://discord.gg/xS7Z362)
is probably the best place to ask for help.

# Front Matter

Before I get into the heart of the matter, I will address some miscellaneous topics that are important to triaging in general.

## Do We Need a Reduction?

Often a bug report will provide a
[Minimal Reproducer Example](https://stackoverflow.com/help/minimal-reproducible-example) or otherwise known as a [SSCCE](https://www.sscce.org/)
via a godbolt link. For clang frontend bugs we also require no standard headers for reproducers (since after preprocessing
standard headers will result in a large amount of code). Often they do not have a minimal reproducer: it might
be a link to repository, a compressed set of code or a godbolt link to code that is not minimal in some way. Once we confirm
it is actually a bug or we can't make progress we usually require a minimal reproducer to make progress. Sometimes it is
trivial to reduce by hand but usually not and it requires the use of tools such as [creduce](https://github.com/csmith-project/creduce) and [cvise](https://github.com/marxin/cvise)
and we suggest using such in the [LLVM tools for folks submitting bugs section](https://llvm.org/docs/HowToSubmitABug.html).

Often labeling the bug with `needs-reduction` is sufficient and one of several folks in the community will pop in and reduce
the bug but we welcome more folks who wish to contribute to reducing bugs. If this is your first time reducing a bug I would
suggest joining the clang discord channel first and coordinate with others so your not duplicating work.

## Bisecting

Often after a new release we will have regressions. Sometimes based on the problem we can determine which change likely
created the regression but if not then we can usually use [git bisect](https://llvm.org/docs/GitBisecting.html) to help
track down the specific commit that led to the regression. Bisecting is useful in general and not just after a new release
but the longer the timespan the longer the bisect will take.

## Formatting Code and Backtraces

Sometimes bug reports will have code and backtraces that are not quoted. We often see this when looking at really old bug
reports. This makes the bug report harder to read and understand. Code and backtraces should be quoted properly, see [Quoting code](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#quoting-code) for details. Usually we will be using ```  `​``cpp ``` or ``` `​``c ``` for C++ and C code and ``` `​``console ``` for backtraces.

## Why We Need Assertions, Backtraces, oh my that is a lot of text

Adding assertion messages, unreachable messages and backtraces can mean a lot of additional text in a bug report. One may
wonder why we need all of that when if we have a godbolt link it will all be there? The long and the short of it is that we
want to be able to find related bugs, especially duplicates. Folks may have seen the same bug before or a related bug. There
may have  been detailed investigation already. We want to avoid redoing the same work and would rather close the new bug as a
duplicate and add any additional information we find the original bug. Additionally linking duplicates may provide a simpler
reproducer for an investigator to use and perhaps speed up finding a fix. Additionally a new investigating may be able to make
significant progress based on previous work done by others and new bug reports often motivate folks to do additional digging.

Linking related bugs also allows us to build a more robust test suite. When fixing a bug we should always add tests to verify
the fix and to assure that we catch any regressions. Being able to find all the related bugs often means we will be able to
add more relevant tests when a fix it committed. A larger variety of tests should catch more bugs in the future.

Finding duplicates also allow for better prioritization. If we are seeing the same bug come up over and over again, it is
likely that bug is a priority to fix.

## Why use Assertions build for Crashes?

User reporting crashes will often not do so against an assertions build. Assertions build will often fail earlier and the
assertion itself usually provides indications to the actual underlying problem. Assertion purpose is to document invariants
and often a violation of an invariant can point to bad assumptions or not rigorous enough checking at earlier points.
Assertions build will also enable `llvm_unreachable` which indicate what should be impossible states. If we are reaching an
impossible state we have done something wrong.

## diverges-from labels

There are three major C and C++ implementations that clang is often compared to: `EDG`, `GCC` and `MSVC`. Outside of vender
extensions, *implementation defined behavior*, *undefined behavior*, *ill-formed no diagnostic required* cases and not yet
implemented features, conforming C and C++ compiler should behave the same way for identical code. When we have divergence in
behavior with other these other compilers we add the `diverges-from` label for each compiler we diverge from.

## Good First Issues

As a project clang and LLVM in general want to provide as many opportunities for new contributors to get their feet wet and
contribute. When we find an issue that we believe a new contributor should be able to tackle without a great deal of
experience we will label these bugs `"good first issue"`. If you apply this label to a project it is encouraged to leave some
guidance in the bug report to help them get started. For example pointing out where in the source code the relevant fix should
take place and acceptable approaches. New contributors may not have commit access nor be members of the project yet. So they
may ask you to assign the issue to them and or review their PR and eventually merge it.

# How To Triage

## Triaging crashes

Probably the more straight forward bugs to triage are crashes. The compiler should never crash, it is always a bug and the
symptoms are usually quite obvious, a backtrace with or without an assertion or less often some other crash without a
backtrace.
There are a few key things we want to do here:

- Determine if we need a reduction and if so add `needs-reduction` label to the bug. At this point you are usually done because we usually can not do further triaging without a reduction.
- Validate you can replicate the crash, preferably on godbolt. If the report has not provided a godbolt link with the reproducer then add the reproducer to the bug report. If we can validate the crash we can label the bug as `confirmed`.
- Once we have a reproducer, make sure to run it with an assertions build and base backtrace on assertions build.
- Note the assertion if any in the bug report.
- Note any unreachable errors in the bug report.
- Note the backtrace in the bug report.
- Note if the crash is a regression and if so which version of clang it started in. If this is a regression on trunk then add the `regression` label and if it a regression against the last release then for example if the last release was clang-19, add `regression:19` label.
- Determine if this is a crash on valid or crash on invalid and label it accordingly.
- If the code was generated via a fuzzer please label it `clang:frontend:fuzzer`
- Once we confirm the crash label the bug `confirmed`.

Some examples of triaged bugs:

- [Example 1](https://github.com/llvm/llvm-project/issues/110775), here we have a crash where the author of the issue added the assertion and the backtrace already so no need to add it. The bug is a crash on invalid, fuzzer generated and has those appropriate labels. I note in the comments is a regression but from a pretty old clang version 14.
- [Example 2](https://github.com/llvm/llvm-project/issues/72880), here the user just provided a zip file. Triagers later on added the minimal reproducer code, assertion and backtrace.
- [Example 3](https://github.com/llvm/llvm-project/issues/110914), here we have a crash that is a recent regression against clang-19. The user did not run against an assertions build so added a assertion and new backtrace based on the assertions build.

## Triaging Undefined Behavior bugs

We won't see these often and without being pretty intimately familiar with most of the *Undefined Behavior (UB)* in *C*
and *C++* there are hard to spot and if you are they are still not easy to spot. Some possible indications that a bug is
really the product of *UB* are

- The behavior changes based on optimization level, sometimes only at `-O2` and `-O3`.
- The behavior changes when using `-fno-strict-aliasing`
- When used in a [constant expression context](https://shafik.github.io/c++/undefined%20behavior/2019/05/11/explporing_undefined_behavior_using_constexpr.html) the code is treated as ill-formed because of *undefined behavior*.

If you do find that the unexpected behavior is due to *UB* then:

- Label the bug `undefined behavior`.
- Explain that the code invokes *UB* and if possible point out alternative methods to achieve the same result without *UB*.

Often these bugs will lead to further detailed discussions. Either because there is some perceived opportunity to diagnose this
better, there is an optimization that may be aggressive and leading to unintuitive or perhaps potentially unsafe behavior or
there is a *Quality of Implementation (QoI)* argument that we can handle this in a better way.

Some examples of *UB* bugs are:

- [Example 1](https://github.com/llvm/llvm-project/issues/109396), here we have an alternative flag that can be used to avoid the UB.
- [Example 2](https://github.com/llvm/llvm-project/issues/60622), we have an example of a long discussion around a specific optimization around *UB* and how we can improve the situation. We also see many related bugs linked into the discussion allowing for easy reference point for all related bugs.

## Triaging Diagnostic Bugs

Developers requests around diagnostics vary amongst: reducing false positives, reducing false negatives, splitting or merging
different diagnostics to allow finer grain control, non-conformance, general *QoI* issues or requesting a new diagnostic all
together. There can be a lot of subjectivity around diagnostics and usually we want one of the maintainers to weigh in on
these bugs. We should:

- If the issue is about conformance then the `clang:frontend` label is appropriate.
- Otherwise label these `clang:diagnostics` and remove `clang:frontend` if it is there.


## Implementations Don't Agree on the Code

It is not uncommon to see bugs in which multiple implementations do different things with the same code. Usually this is an
indication of a bug in one or more of the implementations and the goal of the bug is to determine if clang is doing the right
thing™️. Often these bugs will require consulting the [latest draft C++ standard](https://eel.is/c++draft/) or a specific past
[version of the standard](https://github.com/timsong-cpp/cppwp) and being able to support and argument that clang is correct
or not based on specific sections of the standard. Sometimes the standard is not clear or has conflicting sections that are
relevant to the code in question. This will require the input of the relevant language lawyers and potentially a conformance
code owner. When the standard is not clear this usually result in a language lawyer filing a language issue with the standards
body or writing a proposal to address the issue. Usually in the case of issues brought up with standards body links to the
relevant discussions (which may not be public) or issues will be linked into the discussion.

- If there is a minimal reproducer then add a godbolt link covering each implementation to the bug report.
- If clang is correct (not a bug), add commentary explaining why clang is correct and close the bug as `not planned`.
- If clang has a bug, add commentary justifying why clang is incorrect and label the bug `confirmed`.
- The standard is not clear or is conflicting, then usually we need to CC one of the language lawyers to give an assessment.
- Add appropriate `diverges-from` labels.

Some example bugs:

- [Example 1](https://github.com/llvm/llvm-project/issues/102467), here we have a detailed discussion involving, proposals and defect reports.
- [Example 2](https://github.com/llvm/llvm-project/issues/111622), here we have an example in which the correct answer is not obvious.


## Extensions

Clang and other implementations often implement language extensions. In many cases implementations support the
extensions of each other, for example clang often supports
[GNU extensions](https://gcc.gnu.org/onlinedocs/gcc/C-Extensions.html). Sometimes clang's behavior does not match the
behavior in another implementation.

- Label the bug `extension:gnu` or `extension:microsoft` for GNU or Microsoft extension respectively.

Some example bugs:

- [Example 1](https://github.com/llvm/llvm-project/issues/95936), here we have a case where clang is accepting an extension in more places (likely incorrectly) than gcc.
- [Example 2](https://github.com/llvm/llvm-project/issues/59511), here is one where we are being asked to support an MSVC extension, with a lot of discussion.

## Conclusions

This ended up being a much longer writeup then I intended but it is I feel a pretty complete picture of what triaging
clang frontend bugs entails. This ended up being longer because a lot of what we do in triaging bugs is wrapped up in
good practices for understanding bugs in general. So even if you don't end up participating in LLVM bug triaging I believe
a lot of what I discussed here in generally applicable and good advice.

### Thanks

I would like to thank Aaron Ballman and Vlad Serebrennikov for their feedback on this.
