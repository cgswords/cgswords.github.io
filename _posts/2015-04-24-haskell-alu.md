---
layout: post
title: A Small ALU in Haskell, Part I
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
using Haskell's `&&` and `||` operators.

{% highlight haskell %}
halfBit :: Bool -> Bool -> Bool
halfBit a b = (a && not b) || (b && not a)
{% endhighlight %}

Observe, however, that this adder is equivalent to using `xor`, so
we can simply define it directly. We also include an infix version,
`*|`, for ease of use.

{% highlight haskell %}
xor :: Bool -> Bool -> Bool
xor False False = False
xor False True  = True
xor True  False = True
xor True  True  = False

(*|) :: Bool -> Bool -> Bool
a *| b = a `xor` b
{% endhighlight %}

Finally, we will construct a bit-adder with a carry-out from our
half-bit adders (using `*\`):

{% highlight haskell %}
bac :: Bool -> Bool -> Bool -> (Bool, Bool)
bac a b c = let ab = (a *| b)
            in (ab *| c, (a && b) || (ab && c))
{% endhighlight %}

We are almost ready to construct our ALU, but we have one
last detail: we need some mechanism to encode and decode
natural numbers to binary, representing a number a as a
list of Booleans of length 32:

{% highlight haskell %}
natToBits :: Int -> [Bool]
natToBits 0 = []
natToBits n = if even n
              then False : natToBits (div n 2)
              else True  : natToBits (div (n - 1) 2)

binToNat :: [Bool] -> Int
binToNat     [] = 0
binToNat (x:xs) = if x 
                  then (binToNat xs * 2) + 1 
                  else binToNat xs * 2

padTo32 :: [Bool] -> [Bool]
padTo32 ls = let len = length ls
             in if len < 32
                then ls ++ replicate (32 - len) False
                else ls

natToBin :: Int -> [Bool]
natToBin = padTo32 . natToBits
{% endhighlight %}

Next, we construct our entire ALU in one "fell swoop", building a
simple 
[ripple-carry adder](http://en.wikipedia.org/wiki/Adder_%28electronics%29#Ripple-carry_adder),
to sum two numbers:

{% highlight haskell %}
bitsAdd :: [Bool] -> [Bool] -> Bool -> [Bool]
bitsAdd [] [] c = []
bitsAdd (x:xs) (y:ys) cin =
  let (a, cout) = bac x y cin
    in a : bitsAdd xs ys cout

aluAdd :: [Bool] -> [Bool] -> [Bool]
aluAdd a b = bitsAdd a b False
{% endhighlight %}

This ALU starts with an empty carry-in and iteratively performs `bac`
over each pair of inpus, threading the new carry bit through to the
next computation until we are at the end. Notice that we throw the
final carry bit away; overflowing is just find on a processor.

Even so, we have now constructed a 32-bit unsigned integer adder
ALU from first principles: from `and` and `or`, or, perhaps
more accurately, `and`, `or`, and `xor`. This ALU isn't particularly
useful right now, but in the next installment we will add a multiplexer
and additional ALU operations, then add register store and loads to
support larger computations. 
