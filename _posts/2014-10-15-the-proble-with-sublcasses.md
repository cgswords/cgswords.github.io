---
layout: post
title: The Curious Case of Subclassing and Scope
tags: blog code
---
The following is a brief discussion of classes and the issues with subclassing. 
It was originally written with the intention of explaining thr trouble to
some of my office-mates.
I've become more interested in a reasonable resolution to the dynamic
dispatch problem (which, to be accurate, has two well-defined solutions that
I find unsatisfactory).  I've recently read the 
[Classboxes](http://scg.unibe.ch/archive/papers/Berg03aClassboxes.pdf)
proposal, which I find grossly unsatisfactory (after partially implementing
the semantics, but that's another post), and I decided to look into the
problem.

<hr />

First, let's suppose we're creating objects in a language like Racket. We can
start by defining a small object that has a subobject:

{% highlight scheme %}
(struct self1 (odd even))

(define obj1
  (letrec
      ((odd  (lambda (x) (if (zero? x) #f (even (sub1 x)))))
           (even (lambda (x) (if (zero? x) #t (odd (sub1 x))))))
          (self1 odd even)))

(define subobj1
  (self1 (lambda (x) #t) (self1-even obj1)))
{% endhighlight %}

The first defines a set of functions and constructs an object that
has the two functions (called *methods*) that are mutually recursive. The
sub-object defines a new object that inherits `even` from `obj1` but
redefines `odd` to always return true. What happens if we invoke these
functions?

Let's consider this:

{% highlight scheme %}
> ((self1-odd obj1) 5)
#t
> ((self1-even obj1) 5)
#f
> ((self1-even subobj1) 5)
#f
> ((self1-odd subobj1) 5)
#t
{% endhighlight %}

That worked as we hoped! But why? Because we've cleverly made odd look "the
same" in this example and only fed it odd numbers. Let's consider another call:

{% highlight scheme %}
> ((self1-odd subobj1) 4)
#t
{% endhighlight %}

Let's try defining a new define another subclass: 

{% highlight scheme %}
(define subobj2
  (self1 (lambda (x) 5) (self1-even obj1)))
{% endhighlight %}

Let's try the same calls:

{% highlight scheme %}
> ((self1-odd subobj2) 5)
5
> ((self1-even subobj2) 5)
#f
{% endhighlight %}

Why did that work? Well, 'work'? Because of Lexical Scope: because we've
decided to close over odd in our first definition of even. But objects don't
really work this way. Let's add self!

{% highlight scheme %}
(struct self2 (odd even))

(define obj2
  (letrec
    ((odd  (lambda (self x) (if (zero? x) #f ((self2-even self) self (sub1 x)))))
     (even (lambda (self x) (if (zero? x) #f ((self2-odd self)  self (sub1 x))))))
    (self2 odd even)))
{% endhighlight %}

This is how it works in "modern languages". Now let's see our subclassing:

{% highlight scheme %}
(define subobj3
  (self2 (lambda (self x) 5) (self2-even obj2)))
{% endhighlight %}

Now, some more calls:

{% highlight scheme %}
> ((self2-odd subobj3) subobj3 7)
5
> ((self2-even subobj3) subobj3 7)
5
{% endhighlight %}

And boy is that a problem! We're relying on Dynamic Scope to do what we'd
like here. What's worse, we could imagine that odd is a private function,
which means a subclasses might shatter security:

{% highlight scheme %}
(struct self3 (private-key encrypt encrypt-with-private-key))

(define secure-obj
  (letrec
    ((key 11)
     (encrypt (lambda (self x key) (expt x key)))
     (encrypt-with-private-key
      (lambda (self msg) ((self3-encrypt self) self msg key))))
    (self3 key encrypt encrypt-with-private-key)))
{% endhighlight %}

Now we can imagine encrypt and key are private here, right? And safely call
`encrypt-with-private-key`:

{% highlight scheme %}
> ((self3-encrypt-with-private-key secure-obj) secure-obj 3)
177147
{% endhighlight %}

But a subclass will allow me to only rewrite anything I please, and scope it
the way I want! Note I am calling getters here, but that's what any subclassing
compiler is going to do, anyway.

{% highlight scheme %}
(define secure-subobj
  (self3
    (self3-private-key secure-obj)
    (lambda (self x key) key)
    (self3-encrypt-with-private-key secure-obj)))
{% endhighlight %}

Now I can try to encrypt again!

{% highlight scheme %}
> ((self3-encrypt-with-private-key secure-subobj) secure-subobj 3)
11
{% endhighlight %}

While this isn't really 'a serious error' in some sense, what have we
reconstructed? The ability to arbitrarily perform superclass reflection
via subclassing! And in a language where functions are first-class, I
might even capture private methods defined on a class using a subclass's
definition in this model.

Okay, so this has been problematic, because of Dynamic Scope. Let's try it
the other way, so that we maintain lexical scoping.

{% highlight scheme %}
(define obj3
  (letrec
    ((key 11)
     (encrypt (lambda (x key) (expt x key)))
     (encrypt-with-private-key
     (lambda (msg) ((self3-encrypt self) msg key)))
     (self (self3 key encrypt encrypt-with-private-key)))
    self))
{% endhighlight %}

Self is no longer exposed to the user, and our subclass utterly fails to be
tricky:

{% highlight scheme %}
(define subobj4
  (self3
    (self3-private-key obj3)
    (lambda (x key) key)
    (self3-encrypt-with-private-key obj3)))

> ((self3-encrypt-with-private-key subobj4) 3)
177147
{% endhighlight %}

There's our safty back! We can no longer change the underlying definition
of any method or field in the superclass using this model. 

But what have we lost? Local self-reference!

{% highlight scheme %}
(define subobj5
  (self3
    13
    (self3-encrypt obj3)
    (self3-encrypt-with-private-key obj3)))
{% endhighlight %}

Dang, now we can't change our keys out!

{% highlight scheme %}
> ((self3-encrypt-with-private-key subobj5) 3)
177147
> ((self3-encrypt-with-private-key obj3) 3)
177147
{% endhighlight %}

So it's not actually enough to specify things how we've done: neither lexical
*or* dynamic scope is quite what we want. We'd like to choose ones with some escape
mechanism, which is at least partially-described in Dan Friedman's
[Object-Oriented Style](http://www.cs.indiana.edu/~dfried/ooo.pdf). 
Even so, the paper provides the following discussion of scopeis:
"This decision is quite arbitrary, but any reasonable characterization of 
 which variables shadow which over variables can be [easily implemented]."

Most modern language pick one solution (like explicitly requiring `super`, or 
implicity shadowing things). Unfortunately, there is significant trade-off in
every case, and I can't help but think there's a better solution...
