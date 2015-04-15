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

<div>
Suppose $e \sqsubseteq e'$ and $\cdotp \vdash e : \tau$.
</div>

While interesting in its own right, ...


