---
layout: post
title:  Writing Christmas Trees, or Building A Cute Turing Tarpit
tags:   blog language code
---

My wife and I recently decided to enter this month's contest over at PLT Games.
We produced a variation of the lambda calculus called 
[Tannenbaum](https://github.com/cgswords/tannenbaum) in which every program
looks like (or hopefully looks like) a Christmas tree. For example,
here is [omega](https://github.com/cgswords/tannenbaum/blob/master/omega.tbm),
which is normally written `((lambda (x) (x x )) (lambda (x) (x x)))`. It's just
a cute little bush!

To be honest, the language interpretation itself isn't difficult, once in the
proper form---once everything is lambdas and application tags and variable and
line references. But properly stripping it apart is a fancy trick. Luckily,
most of the string processing tools I needed got written while I was writing
that blogging software last week. The harder part, though, was properly picking
apart application, and that's the algorithm I'm going to spend this talking
about.

The idea is simple: given application tags and the guarantee that every
application is of one argument, properly group up the arguments, separated only
by `%`. For example, `(@ @ $ $ % $ % $ $ $)` is, in Scheme, 
`(((var 1) (var 0)) (var 2))`. But given a chain of application tags and
delimiters, how does one group up and properly partition these?

I wouldn't call my solution elegant, but it gets the job done. I have never
done this before, so it was certainly a first for me. (I imagine that automatic
currying follows a similar pattern, so maybe this is a more useful exercise
than I initially suspected.) But I've pontificated enough; onto this algorithm:

The algorithm I have uses three helpers: `app-splitter`, `app-builder`, and
`app-parser` (which really just calls `app-builder` on `app-splitter`'s output). The
`app-splitter` function walks over the input, as shaped above, using two stacks.
Any time it sees a symbol such as a `+`, `$`, or `0`, it simply pushes that symbol
onto stack A. On a `%`, the current contents of stack A are moved to stack B as
their own element (as a list). This means that the delimiters chunk up the
input.

When an `@`---application---tag is found, if stack A is empty an app tag is
pushed onto stack B. If stack A is empty, it is moved onto stack B before the
app tag is pushed on. At the end of all of this, the string is split into
tokens based on its application and argument delimiters, but still shaped
incorrectly (and backwards).

{% highlight scheme %}
(define app-splitter
  (lambda (ls stack1 stack2)
    (cond
      [(null? ls) (cons stack1 stack2)]
      [(eq? (car ls) '+) 
        (app-splitter (cdr ls) (cons '+ stack1) stack2)]
      [(eq? (car ls) '0) 
        (app-splitter (cdr ls) (cons 0 stack1)  stack2)]
      [(eq? (car ls) '$) 
        (app-splitter (cdr ls) (cons '$ stack1) stack2)]
      [(eq? (car ls) '%) 
        (app-splitter (cdr ls) '() (cons stack1 stack2))]
      [(and (eq? (car ls) '@) (null? stack1))
       (app-splitter (cdr ls) '() (cons 'app stack2))]
      [(eq? (car ls) '@)
       (app-splitter (cdr ls) '() (cons 'app (cons stack1 stack2)))])))
{% endhighlight %}

This moves us to app-builder, which takes this reversed list and adds proper
nesting by using its own stack as follows: it 'pops' something off the original
stack handed to it (stack B from before), and if it's not an app tag, the
element gets pushed onto app-builder's own stack C. An app tag pops the
previous two elements off of stack C and pushes the whole 
`(app [element 1] [element 2])` back onto the stack (in case it's nested in
another application tag). This repeats until stack B is empty, at which point
(if the input was correct), stack C contains the properly-shaped result.

{% highlight scheme %}
(define app-builder
  (lambda (ls stack)
    (cond
      [(null? ls) stack]
      [(eq? (car ls) 'app)
        (app-builder
          (cdr ls)
          `((app ,(car stack) ,(cadr stack)) . ,(cddr stack)))]
      [else (app-builder (cdr ls) (cons (car ls) stack))])))
{% endhighlight %}

This completes the parsing, handing out the result. I thought this approach was
neat (though possibly a little expensive); it requries two passes over the code
for O(2n) complexity based on input size. It is certainly possible to do all of
this in one pass, but it would require performing building up tokens while
reshaping applications, and must be done back-to-front.

As an interesting aside, I find this entire exercise has led me to one
interesting historical insight: I suspect the reason that APL is right-to-left
precedence is for precisely this reason of application. Since this requires far
less backtracking when done in reverse, if a computer with little power is
reading from left to right, providing precedence from right to left allows you
to perform this analysis while reading input.

(It's worth noting that doing this in one fell swoop only requires one stack,
and would probably be far more efficient after the initial reversal. That said,
I feel like this implementation is more clear at the cost of efficiency.)

I hope this article helps someone somewhere; writing a parser like this is a
nice academic endeavor, if nothing else, and I encourage you to write one for
your favorite language some time.

