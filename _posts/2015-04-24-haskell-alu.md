---
layout: post
title: A Small ALU in Haskell, Part I
tags: blog code haskell bits
---
I consider ALUs (Arithmetic logic units) to be interesting because they are a
direct application of Boolean algebra, and it is easy to extend into something
realistically resembling a computer (from the 1980s). This post is going to be
about sketching a small ALU implementation in Haskell. I originally wrote this
code for a discussion section for a Discrete Math honors course, presented
right after they learned Boolean algebra. 

I have since cleaned the code up once or twice, and I decided it would
be neat to present it here. Due to its length, I am going to do it in a
few parts: 

1. We will construct a simple ALU Adder for 32-bit Boolean values 
   Next, we will construct a larger ALU to support `and`, `or`, and
   `xor` and then construct a multiplexer to give us the correct
   answer.  
2. The second installment will also feature the addition of registers
   with `load` and `save` to feature more general computation. After,
   that we will add program counter, branching and jumps to construct
   something resembling a full computer.

In this first installment, we begin simply with the classical
definition of `xor`, which will allow us to circumvent the classical
construction of a 
[half-bit adder](http://en.wikibooks.org/wiki/Digital_Circuits/Adders#Half_Adder),
We also include an infix version, `*|`, for ease of use. This
implementation should be straight-forward; we simply dispatch on the
input and produce the appropriate truth value.

{% highlight haskell %}
xor :: Bool -> Bool -> Bool
xor False False = False
xor False True  = True
xor True  False = True
xor True  True  = False

(*|) :: Bool -> Bool -> Bool
a *| b = a `xor` b
{% endhighlight %}

We are now ready to construct a whole-bit adder from two half-bit adders
with a bit of "glue" to connect the three inputs with the two outputs.
I would refer the interested reader to [this
page](http://en.wikibooks.org/wiki/Digital_Circuits/Adders#Full_Adder)
to see how this works with circuitry; the idea is that we add the
three bits through two half-bit adders, and then do an appropriate
amount of math to construct the carry-out bit (that is, the carry that
results from the addition):

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
to sum two numbers. This implementation will take both inputs as lists
of Boolean values and iterate over them, rippling the carry bit
through to compute the binary addition in terms of `bac`.

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
useful right now, but extending it to feature a multiplexer is
straight-forward. We will "cheat" here for simplicity, however.
Constructing a full multiplexer from `and` and `or` requires a series
of additional analog circuitry:

![Analog Multiplexer]({{ site.url }}assets/images/multiplexer.gif)

This construction is left as an exercise to the reader; instead,
we will opt for the following digital multiplexer, and define
our operations in terms of it.

{% highlight haskell %}
multiplexer :: [[Bool]] -> [Bool] -> [Bool]
multiplexer ans [False, False] = map (!!0) ans
multiplexer ans [False, True]  = map (!!1) ans
multiplexer ans [True, False]  = map (!!2) ans
multiplexer ans [True, True]   = map (!!3) ans

ops = [("+",   [False, False]),
       ("and", [False, True]),
       ("or",  [True, False]),
       ("xor", [True, True])]
{% endhighlight %}

The output of our ALU now constructs a list of lists, where each inner list is
the result of each operation on the appropriate bits, where the first is
addition, the second is `and`, the third is `or`, and the fourth is `xor`.  We
select each one out, and construct that result as our final computational
result for the operator. This requires a modification to our ALU, however, to
return this shape in every step:

{% highlight haskell %}
bitsALU :: [Bool] -> [Bool] -> Bool -> [[Bool]]
bitsALU [] [] c = []
bitsALU (x:xs) (y:ys) cin =
  let (a, cout) = bac x y cin
  in [a, x && y, x || y, x *| y] : bitsALU xs ys cout
{% endhighlight %}

Then our final ALU uses the multiplexer on the result of this bit ALU to build
the final computational ALU we will use:

{% highlight haskell %}
alu :: [Bool] -> [Bool] -> [Bool] -> [Bool]
alu a b = multiplexer (bitsALU a b False)
{% endhighlight %}

We now have something resembling a computer! But it is not enough to make this
claim, so we will demonstrate that it is the case. To this end, we construct
a small language for our ALU, with a simple test case:

{% highlight haskell %}
data Op = Op String [Bool]
data End = End
data Program = Start [Bool] [Op] End

t1 = Start
       (natToBin 2)
       [Op "+" (natToBin 4),
        Op "and" (natToBin 6),
        Op "or" (natToBin 8)]
     End
{% endhighlight %}

Finally, we build a small driver that peforms these programs,
using our previous definition of operations to look up and
dispatch to the multiplexer appropriately:

{% highlight haskell %}
runALU1 :: Program -> [Int]
runALU1 (Start input prog End) = runHelper input prog
  where
    runHelper input [] = toInts input
    runHelper input (Op s b:prog) =
      case lookup s ops of
        (Just opcode) -> let output = alu input b opcode
                         in runHelper output prog
        (Nothing)     -> error $ "Invalid operation " ++ s
{% endhighlight %}


{% highlight haskell %}
*Circuits > runALU1 t1
[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,1,0]
{% endhighlight %}

The output is 14, which, it turns out, is the result of `(2 + 4) && 6 || 8`
(unsurprisingly). We now have a basic language---more of a calculator, but
still! We can perform basic Boolean logic operations over our input, which
lives in the "operator" register. But still, we've constructed the basics
of computation out of `and`, `or`, `xor`, and small operator dispatch using
a multiplexer, and built a language on top of it.

In our next installment, we will extend our language and framework with
full registers and then branching operators, constructing a small
instruction set for general computation on top of this ALU.  
