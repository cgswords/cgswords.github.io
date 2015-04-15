---
layout: post
title: The Refined Gradual Guarantee and Compilation
tags: blog code gradual compilation
---
There was some recent work by some of my colleagues toward a 
[Refined Criteria for Gradual Typing](https://dl.dropboxusercontent.com/u/10275252/gradual-guarantee.pdf),
wherein they propose that current many modern gradual typing systems are not
faithful to the original aim of gradual typing. In particular, the following
assertion is at the heart of their refined criteria:

    ...programmers should be able to add or remove type annotations without any
    unexpected impacts on their program, such as whether it still typechecks and
    whether its runtime behavior remains the same.

This criteria is ultimately rephrased as a new theorem-shape, called the
*gradual guarantee*, as follows:

Suppose $e \sqsubseteq e'$ and $\cdotp \vdash e : \tau$. Then:

1. $\cdotp \vdash e' : \tau'$ and $\tau \sqsubseteq \tau'$
2. If $e \Downarrow v$, then $e' \Downarrow v'$ and $v \sqsubseteq v'$, and
   if $e \Uparrow$, then $e' \Uparrow$.
3. If $e' \Downarrow v'$, then $e \Downarrow v$ where $v \sqsubseteq v'$ or 
   <span>$e \Downarrow \mathsf{blame}_\tau l$</span>, and
   if $e' \Uparrow$, then $e \Uparrow$ or 
   <span>$e \Downarrow \mathsf{blame}_\tau l$</span>.

While interesting in its own right, I think that this guarantee can be extended
in an interesting and insightful way: instead of using this as a criteria for
describing the interactions between the typed and untyped portions of a
gradually-typed language, we might refocus this to help encode the process of
compiling from typed to untyped languages.

**Aside:** *Before I continue, I should point out that I am unfamiliar with the
current work in type-safe compilation. If such a criteria has already been 
explored for compilation, I'd love to see it!*

While still sketchy, I think that it should be straight-forward to develop a
sort of "gradual" guarantee for the compilation process as well. I will
further break this idea out in the three components of the theorem.

First, we take $e$ and $\tau$ to be the *input* to the compiler (or to a pass or
sequence of passes) and $e'$ to be the output. While compilers may discard the
strong types early, we can still reason about the "simplistic" types that the
pass works over, whether it is expression forms or explicit memory layouts.

The first criterion suggests that if the original expression is "well-typed",
than the result of compilation must yield something that is also well-typed.
Furthermore, this result is "compatibly" typed: it is equivalent, but may be
less precise, than the original type.

The second criterion suggests that if, under our operation semantics, the
original expression yields some value $v$, then after compilation our result
should yield some value $v'$ that is less precise, but otherwise equivalent,
to the original expression.

The third criterion provides a more interesting insight: it constrains what
a compiler may produce. We take this third standard to suggest that if
the result of a compiler (pass) terminates in a value, this result is
only reasonable if there is some input expression that is more precise.
Moreover, this input expression may produce errors where the output
does not, but is otherwise behaviorally identical.

This last criterion is, in some sense, the crux of the "risk of" compilation:
our compiler may transform code with errors into code that does not produce
these errors by removing critical information during the compilation process.
This is the difference, in C, between producing a type error and accidentally
jumping into and executing an integer-as-pointer. 

While potentially problematic in terms of program meaning, this is a necessary
horror of compilation: a compiler may take a program that, under the original
semantics, should produce an error, and (possibly incorrectly) conceal this
error.

