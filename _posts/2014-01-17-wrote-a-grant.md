---
layout: post
title: I Wrote a Grant
tags: blog research
---
So I wrote a grant. It wasn't particularly good, and it isn't
likely to get funded, and it got finished in *far too little time* to be
shiny, but it got written. Some day's that's all we can ask for. The
experience was intense, and amazing, and it yielded some interesting
insights and ideas. I've decided to reprint the overview here, for those
who are interested or otherwise curious.

**Modular Compilers using Dependent Effect Handlers**

The use of monads and monad transformers for constructing modular programs
is now well-established. In this context, a "modular program" is an
*extensible* program constructed from reusable blocks. This technology was
first demonstrated and popularized for *interpreters*, but it has proven
difficult to extend this approach to compilers.  

Recent advances in combining computational effects using *effect handlers*,
in the implementation of *dependent types*, and compiler architectures such
as the nanopass framework provide new opportunities that we may exploit to
design and implement modular composable compilers. Our main objective is to
refine this theory and adapt it to write a full compiler for the
dependently-typed language, Idris.

Idris is currently implemented in Haskell, providing a number of features,
including fully dependent types, type classes, dependent records, pattern
matching, a tactic-based theorem prover, and totality checking. We propose
to develop a self-hosting compiler for Idris from the type-checker to the
executable generator, building the necessary language facilities along the
way. Once complete, we will harness the dependent type system of Idris to
organize the compiler in a collection of composable and extensible modules
that can be used to prove language properties, or to perform aggressive,
type-guided optimizations including novel non-standard analyses and
transformations related to information-flow and energy consumption.

*Wish me luck!*
