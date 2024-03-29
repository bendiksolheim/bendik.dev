+++
title = "Characteristics of the Perfect Programming Language"
date = 2020-12-02

+++

This is an opinionated post. Consider yourself warned. You will probably disagree with at least some of it, and that is completely fine. The important part is the thought process – don’t just read through it and agree or disagree, think about what you agree or disagree with and why. Becoming aware of these things is valuable: once you become aware of why you dislike something, you can start doing something about it.

<!-- more -->

Over the years, I have noticed there is a small set of features I value more than others in programming languages. These features allow me to express myself in a concise and clear manner, makes my code more readable, and minimizes the amounts of errors I make. They are not "magical" in any way. Far from it. Rather, they are more.. fundamental aspects of the progamming language itself. Let’s dive in!

## Expressive type system

Types help me reason. Explicitly typed code communicates intent better, and is easier to navigate without losing context completely. Type systems comes in a variety of flavors, ranging from the most simple ones to more advanced systems with gradual typing, dependent types, type classes, and so on. Although I personally prefer more advanced ones, I believe this is more of a subjective topic. And of less importance, really. The main benefit of type systems exists in even the simplest ones: increased safety and readability. Remember: the compiler is your friend – the more it can verify, the better!

## Type inference

I was a bit unsure if this point had its place in this blog post. On one hand, I feel its more of a convenience feature than strictly neccessary. On the other, an overly chatty and boilerplaty type system can cause more harm than good – if your business logic drowns in types, it’s too far in the wrong direction. Java, around the 1.5 era, is an example of this. In addition, type inference itself is a a double-egded sword. If the compiler inferes a different type than what you would have thought, it can be quite hard to spot where the actual error is. This is the reason some languages and communities define best practices around type inference, like being explicit in function signatures. Anyway. When I find that perfect spot between explicit types and inferred types, it just makes me really happy.

## Algebraic data types

I remember the first time I enountered sum types. My first response was «but.. Isn’t this just some kind of enums?». I can kind of see why I thought that, but I feel this is a limiting way of approaching them. Where enums helps me limit valid values, algebraic data types is so much more. It enables me to model my domain and state in a precise and correct manner, and makes illegal states «impossible». This didn’t really become clear to me until I went back to Java. I find modelling without them cumbersome and hard, and sometimes even end up in situations where I can’t avoid impossible states. Few things are more confusing than finding the comment «this can’t happen» in a branch of someones code.

## First class functions

I don’t even understand why this is subject to discussion anymore, but functions should be treated just like any other value [^1] in a programming language. Returning functions and accepting functions as parameters is a fundamental feature to create abstractions and solve problems with composition. This just isn’t up for debate – I would take first class functions over any other feature any day of the week!

## Functional programming

Functional programming taught me to value these features. Sure, you will find imperative languages with these features as well, but they never feel like a core feature of the language. Even Java comes with some sort of primitive type inference these days, but it mostly feels tacked onto the language.

With some notable exceptions (looking at you, LISP), these are core features in all functional programming languages I use. They make my day better, so I would think hard before adopting a language without them these days.

Which feature do you miss or disagree with the most?

---

[^1] _I use the term "value" loosely here. I don’t mean to debate value types, reference types, and pass by value or reference. Those are implementation details. The point is to see what advantages you get when a language treats simple values and functions the same._
