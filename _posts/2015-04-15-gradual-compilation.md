---
layout: post
title: The Refined Gradual Guarantee and Compilation
tags: blog code gradual compilation
---
There was some recent work by some of my colleagues toward a 
[Refined Criteria for Gradual Typing](https://dl.dropboxusercontent.com/u/10275252/gradual-guarantee.pdf),
wherein they propose that many modern "gradual typing" systems are not
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

While interesting in its own right, I posit that this theorem-shape is maybe not
particularly new. In the above theorem, $sqsubseteq$ is taken to mean *"e is at
least as precise as e'"*. If we reformulate the theorem, however, using a notion
of compilation as $e' = [[e]]$, we see an old friend:

Suppose $e' = [[e]]$ and $\cdotp \vdash e : \tau$. Then:

1. First, $\cdotp \vdash e': \tau'$ and $\tau' = [[\tau]]$
2. If $e \Downarrow v$, then $e' \Downarrow v'$ and $v' = [[v]]$, and
   if $e \Uparrow$, then $e' \Uparrow$.
3. If $e' \Downarrow v'$, then $e \Downarrow v$ where $v' = [[v]]$ or 
   <span>$e \Downarrow \mathsf{blame}_\tau l$</span>, and
   if $e' \Uparrow$, then $e \Uparrow$ or 
   <span>$e \Downarrow \mathsf{blame}_\tau l$</span>.

This is a fairly classic formulation of compilation (and optimization)
correctness. Software Foundations uses a similar statement as one of the
earliest proofs over any language:

{% highlight coq %}
Theorem optimize_0plus_sound: forall a,
  aeval (optimize_0plus a) = aeval a.
{% endhighlight %}

The revision to criterion one is of particular interest, however: it dictates
that (a) there is a compilation procedure for types and (b) it must do something
reasonable in relation to the language compilation. Criterion two follows the
exactly expected shape for compiler correctness, and criterion three is a
corollary to the second (as observed in the original work). What can we glean
from this formulation?

First, that the interactions between dynamic and static pieces of code can
be thought of as, in some sense, an FFI. Writing dynamic code is almost akin
to writing inline assembly in C: you are essentially demanding that the
language trust you, saying "don't check my work". I think that this is a 
good lens to view dynamic programming through: it's an approach that suggests
programmer power, and a demand for trust from the language evaluator (or
compiler).

Second, motions up and down the "lattice of languages" presented in the paper
seem to have some relation to an expression moving through the compilation process.
The difference, of course, is that the "target" and "source" (that is, dynamic
and static) pieces of gradual typing are both syntactically subsets of the
gradually-typed language.

Third, the corollary, when refocused on compilation, ultimately describes the
lossy nature of compilation. Two or three or five source programs may yield the
same final output after desugaring, optimizations, and compilation, and there
are a dozen others that may contain potentially unsafe regions but otherwise run
to completion. This isn't a surprising result by any stretch, but it raises an
interesting insight into how post-compilation programs may behave in relation
to their pre-compiled counterparts.

Finally, however, this reformulation suggests a strong link between the sorts
of properties we would like to guarantee about compilation and the guarantees
we would like in the interactions between dynamic and static programs in
gradually-typed languages. And this yields a further set of questions:

1. What other properties about compilation do we usually want? 
2. Can we reformulate these properties to work over gradually-typed terms? 
3. Which of these additional properties are meaningful and worthwhile?

I don't have those answers right now, but I certainly think that this warrants
further exploration.
