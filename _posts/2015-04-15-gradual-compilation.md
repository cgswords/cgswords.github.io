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
to the original expression. This criterion states, in effect, that our
compiler behaves correctly.

The third criterion provides a more interesting insight: it constrains what
the target language may do while maintaining a ling to a the source language.
We take this third standard to suggest that if the result of a compiler (pass)
terminates in a value, this result is only reasonable if there is some input
expression that is more precise. Moreover, this input expression may produce
errors where the output does not, but is otherwise behaviorally identical.

The original intent of the third criterion is to facilitate moving from untyped
to typed programs: to take programs that don't have types and add them,
promising that the only change may be type errors. Unfortuately, compilation
is more typically one-way. In the context of compilation, this suggests that
the result of the compiled program will behave as the input program, modulo
some errors that the program may be detectable before compilation. 

Why is it important to note that a compiler might remove errors? What do we gain
from this? I think, at the crux, this ultimately describes the lossy nature of
compilation. Two or three or five source programs may yield the same final
output after desugaring, optimizations, and compilation, and there are a dozen
others that may contain potentially unsafe regions but otherwise run to
completion.

This raises a better question, however: does this criterion buy us anything
in the realm of compilation? Let us reformulate the theorem, discarding the
precision relation in favor of the compilation relation.

Suppose $e' = [[e]]$ and $\cdotp \vdash e : \tau$. Then:

1. First, $\cdotp \vdash e': \tau'$ and $\tau' = [[\tau]]$
2. If $e \Downarrow v$, then $e' \Downarrow v'$ and $v' = [[v]]$, and
   if $e \Uparrow$, then $e' \Uparrow$.
3. If $e' \Downarrow v'$, then $e \Downarrow v$ where $v' = [[v]]$ or 
   <span>$e \Downarrow \mathsf{blame}_\tau l$</span>, and
   if $e' \Uparrow$, then $e \Uparrow$ or 
   <span>$e \Downarrow \mathsf{blame}_\tau l$</span>.

Is there ever a case, in our compilation process, where the second property
will hold but the third will not? The original publication suggests that 
property 3 is a direct corollary to property 2, and I am inclined to agree:
any compiler that behaves correctly and precisely state that its output
matches its input, modulo the opportunity for errors. This isn't, however, a
surprising result: the first two parts of this theorem seem natural gaurantees
for a compilation process. Indeed, one of the first proofs in Software
Foundations regarding languages deals with the second criterion:

{% highlight coq %}
Theorem optimize_0plus_sound: forall a,
  aeval (optimize_0plus a) = aeval a.
{% endhighlight %}

This naturally suggests (perhaps rightly so) that the relation between static
and dynamic languages can be thought of as a sort of compilation process. 
It also asks another question: what other properties about compilation do we
usually want? Can we reformulate these properties to work over gradually-typed
terms? Which of these additional properties are meaningful and worthwhile?
