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
2. If $e \Downarrow v$, the $e' \Downarrow v'$ and $v \sqsubseteq v'$, and
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

***Aside:** Before I continue, I should point out that I am unfamiliar with the
current work in type-safe compilation. If such a criteria has already been 
explored for compilation, I'd love to see it!*

