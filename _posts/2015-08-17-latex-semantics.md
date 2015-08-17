---
layout: post
title: Using LaTeX for Programming Language Semantics
tags: semantics code latex typesetting pl
---
LaTex is a fantastic tool for typesetting, but there seem to be a serious gap
in documentation for using it to lay out programming semantics. Moreover, 
getting started is often opaque, obtuse, or generally just difficult. This
guide is intended to alleviate some of that annoyance, and to describe how
LaTeX can be used to easily and quickly lay out language semantics.

There is a [repository on GitHub]() that contains the LaTex I'm going to be
presenting here if you'd like to follow along.

**Getting Started**

First, you're going to want a few packages. 

- [Ben C Pierce's Rules Layout Commands](http://www.cis.upenn.edu/~bcpierce/papers/bcprules.sty)
- Default Packages: amsmath, listings, amsthm, and amssymb. These should have come with your
  LaTeX install.
- The [xspace](https://www.ctan.org/pkg/xspace] package for layout within prose
- The [stmaryd](https://www.ctan.org/pkg/stmaryrd?lang=en) package for additional symbols (as needed).

These last two may come with your LaTeX install. Check if they do before trying
to install them a second time.

**Defining a Language**

With these packages in place, we'll start our document in the usual way.

{% highlight latex %}
inputBox : Address Action -> String -> (String -> Action) -> Html
inputBox addr ph f =
  input
    [ placeholder ph
    , on "input" targetValue (\ str -> Signal.message addr <| f str )]
    ]
    []
{% endhighlight %}



**Building Semantic Reductions**


**Layout and Presentations**

**Conclusion**

