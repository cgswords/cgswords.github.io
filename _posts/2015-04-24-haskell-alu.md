---
layout: post
title: A Small ALU in Haskell
tags: blog code haskell bits
---
This post is going to be about sketching a small ALU implementation in Haskell.
I originally wrote this code for a discussion section for a Discrete Math honors
course, presented right after they learned Boolean algebra. I consider ALUs
to be interesting because they are a direct application of Boolean algebra, and 
it is easy to extend into something realistically resembling a computer (from the
1980s).

I have since cleaned the code up once or twice, and I decided it would
be neat to present it here. We start simply: we define a half-bit adder
using Haskell's `and` and `or` operators.

{% highlight haskell %}
halfBit :: Bool -> Bool -> Bool
halfBit a b = (a && not b) || (b && not a)
{% endhighlight %}

