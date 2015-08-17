---
layout: post
title: ICFP 2015 - An Elm Crash Course
tags: blog code elm bits
---
This last weekend, I competed in the ICFP 2015 programming contest with my wife.
To make a long story short, I don't expect to win, I may not even place, but the
weekend was an absolute pleasure anyway. Here's 
[our submission](http://riswords.github.io/icfp2015/).

We built the submission in [Elm](http://elm-lang.org), an upcoming language that
boasts strong types, functional-reactive programming, and "blazing-fast HTML".
It's like Haskell and JavaScript got together, built something more convenient
for web programming, and then decided that functions were better than objects.
Programs in it are concise and powerful---that entire submission above is less
than two thousand lines. For a more specific example, here's a definition of
a generic input text box function: 

{% highlight haskell %}
inputBox : Address Action -> String -> (String -> Action) -> Html
inputBox addr ph f =
  input
    [ placeholder ph
    , on "input" targetValue (\ str -> Signal.message addr <| f str )]
    ]
    []
{% endhighlight %}

It takes an address to send actions to, a default placeholder string for the
field, and a callback that takes an input string and sends it along the
address (to a mailbox, which are similar to channels but better fit the
FRP model). It produces an HTML object that can ben the displayed as the
`main` of the application.

Over the course of the weekend, we built a bunch of neat things in Elm:

- A system for managing hexagonal grid structures
- A search algorithm for placing pieces
- A visual front-end for the application (as displayed in the link above)
- A small setup for JSON parsing

While each is complex in its own right, I find the last two of particular
interest: while the other two are artifacts of the competition, both the
JSON parser and the visual front-end demonstrate the power and promise of the
Elm language. In this post, I'm going to briefly explain both, and in the
course, hopefully shed some light on FRP, mailboxes in Elm, and the 
advantage of this approach to web programming.

**The Hex Drawer**

Even the main loop does, actually:

{% highlight haskell %}
main : Signal Element
main = looper emptyModel

looper : HexModel -> Signal Element
looper init =
  Signal.map (viewer box.address)  <|
    Signal.foldp
      (\ (action, time) state ->
        case action of
          Init model  -> RunningGame
                           { initialInfo | startTime <- Time.inSeconds time }
                           model
                           []
          TimeLimit n -> setTimeLimit n state
          Nop         -> updateGame <| updateTime time state)
      (RunningGame initialInfo init [])
      (Signal.map2 (,) box.signal (Time.every 3))
{% endhighlight %}

First, some syntactic explainations: 

- Function Sequencing: Elm, like Haskell, provides the `$` control operator. In
  Elm, however, this is written `<|`. In addition, Elm provides a secondary
  operator, `|>`, that handles the other side (in case you want forward
  associativity). It similarly provides `<<` and `>>` to replace Haskell's `.`
  operator.
- Elm has constructs similar to Haskell newtypes, but that more closely mirror
  JavaScript records. It also provides a convenient updae syntax, where you
  declare the variable your're updating as `{ name | field <- <new value>}`.
  We can see this going on in the `Init` case of the fold.

Now, onto the crux of what's going on: Elm is built, from the ground up,
to work in a functionally-reactive style. Everything works over *signals*,
which can be "folded" over with `foldp`. Folded may be misleading here: unlike
folding over a list, you are folding over a continuous *stream of inputs*.
In our case, the signal consists of two parts: a timer that updates every 3
milliseconds, created with `Time.every 3`, and a mailbox signal taken from
`box.signal`. Mailboxes are concurrent structures that accept messages
and produce signals; in this situation, our mailbox is receiving Action
messages and our main loop is dispatching on them. (Messages are sent to
this mailbox by the interface, such as the button we saw at the start
of this post.)

At each itearction over the signal, we perform the action in the `foldp`: we
examine the action state and then either set a new time limit for the game
state, set up a new game state, or update the current one (via `updateGame`) if
there's nothing else to do.

**The Json Parser**



**In Conclusion**

Elm is a neat language still in its infancy. It has a long way to go, but there
is a lot of promise. It needs work, though: the API is small and somewhat
lacking, but growing rapidly. If you like what you see, check it out. 


