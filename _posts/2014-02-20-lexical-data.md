---
layout: post
title: Lexical Data - Lexical Scoping is a Right, Not a Privilege
tags: blog languages scope
---
I had an interesting conversation with 
[Edward Amsden](http://www.edwardamsden.com/) yesterday. 
The crux of it was his claim that modern functional languages do an
*incredibly poor* job of scoping data definitions. By comparison, languages
like Scheme, ML, and Haskell can easily and cleanly scope *any* value you
give it through `let` and `lambda`. And the variables bound by these forms
don't have a large scope or a long life time: they don't live as a top-level
definition. So why do ADTs?

The language `Idris` introduces name spaces, which help ease the problem,
but they still require programmers to define data types in a
namespace-global way:

{% highlight haskell %}
    namespace Foo
      data Env a where
        Empty : Env a
        Ext : a -> Env a -> Env a
     
      interp : Exp -> Env Val -> Val
      interp = ...
{% highlight haskell %}

It's interesting, though. If I would like to define an interpreter of type
`Exp -> Val` which implicitly uses some environment to find the result
without wishing to expose it to the user, I can easily write the full
version inside in even Haskell:  

{% highlight haskell %}
    interp :: Exp -> Val
    interp e = interpH e []
      where
        interpH :: Exp -> Env Val -> Val
        interpH = ...
{% highlight haskell %}

And yet, the environment here must still be exposed in the global namespace
of the local module:

{% highlight haskell %}
    data Env a = Empty | Ext a Env
{% highlight haskell %}

This is *globally* exposed: not only does `interpH` use it, but any other piece
of code is free to use this environment definition anywhere in the module.
The philosophical argument behind functional programming that that
*functions are first-class values*. So why do we insist that functions
should get this treatment, but data-type declarations do not? 

That's the crux: why can't I also define lexically-scoped data, close to the
way I can define lexically-scoped helper functions? Ideally, I'd like to
write this code:

{% highlight haskell %}
    interp :: Exp -> Val
    interp e = interpH e []
      where
        data Env = Empty | Ext a Env
        interpH :: Exp -> Env Val -> Val
        interpH = ...
{% highlight haskell %}

I want the entire type of `Env` *lexically enclosed*, just like helper
functions are. And further, I'd like for *every definition* to get such
treatment. Even type-class instances should get this! If it'd like, I should
be able to write this code:

{% highlight haskell %}
    interp :: Exp -> Val
    interp e = interpH e []
      where
        data Env = Empty | Ext a Env
        instance (Show Env) where
          ...
        interpH :: Exp -> Env Val -> Val
        interpH = ...
{% highlight haskell %}

It's the case that `Idris` *does* support this, quite nicely. (David
Christiansen has provided this code.)

{% highlight haskell %}
    module Teeeest

    fnord : Nat -> Nat
    fnord z = case Bar of
                Bar => S z
                Something => z
      where data Foo = Bar | Something

    --- REPL
    *Teeest> fnord 5
    6 : Nat
    *Teeest> Foo
    (input):1:1:No such variable Foo
    *Teeest> Bar
    (input):1:1:No such variable Bar
    *Teeest> 
{% highlight haskell %}

It's likely this works in `Agda`, too. And this is *important*; data
definitions should be subject lexical scoping. Why should functions get
preferential treatment?
