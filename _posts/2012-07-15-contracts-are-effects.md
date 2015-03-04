---
layout: post
title:  Contracts are Effects
tags:   research school contracts
---
Contracts are effectful; it's that simple. Checking a contract requires code
evaluation'you have to touch something somewhere in the system to get the
contract checked, even if that's just the program counter.

It's easy to discount this 'program counter' effect. Then again, the subtle
nature of continuations in Scheme allow you to craft a continuation that knows
if its been invoked twice, so it's possible even without a literal program
counter. And if you can detect that you are in a contract, you can behave when
you're in a contract and misbehave when you aren't.

This means, at least in Scheme (or, more specifically, Racket), contracts are
fundamentally broken. Yes, they are implemented using run-time mechanisms, and
yes they are invisible on the stack. But if I can know that a contract touched
my input, I can misbehave only when I'm not being observed. In short, the
meaning preservation promised by most contract systems is a blatant lie. (It is
clear that a Haskell eager contract system violates this constraint by forcing
evaluation, but I think it is significant that it is a general result.)

Reconciling this fact has become a focus of my research with Amr Sabry; we're
looking at languages features including monads and handlers (see Eff) to address
this failing of contracts head-on. By admitting this effect, we hope to find a
way to embrace it. Checking a contract must be an effect, and so we aim to deal
with it as such.

