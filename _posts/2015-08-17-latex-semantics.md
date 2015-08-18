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

There is a [repository on GitHub](https://github.com/cgswords/texsem) that
contains the LaTex I'm going to be presenting here if you'd like to follow
along.

**Getting Started**

First, you're going to want a few packages, including proof, amsmath, listings,
amsthm, and amssymb. These should have come with your LaTeX install.

Now, we start our document in the usual way:

{% highlight latex %}
%% FILENAME: paper.tex

\documentclass[9pt]{article}

\input{defs.tex}

\setlength{\abovedisplayskip}{2pt}
\setlength{\belowdisplayskip}{2pt}

\begin{document}

\end{document}
{% endhighlight %}

Notice the input here: we'll use a definition file to *keep us sane* while
defining new forms and structures for our language. This definition file will
contain *all* of the language form definitions, semantic step definitions, etc.,
organized as a sort of reference guide with easy extensibility.
(It will also perform all of the package requirements we may need.)

The two `displayskip` definitions are for using math display environments (which
we will leverage a lot) without a ton of extra spaces floating around. 

**Defining a Language**

Next, we begin by defining a few helpers for language definition in `defs.tex`.

{% highlight latex %}
%% FILENAME: defs.tex

\usepackage{amsmath, listings, bcprules, amsthm, amssymb, xspace, stmaryd}

%%------------------------------------------------------------------------
%% DEFINITION HELPERS
%%------------------------------------------------------------------------

\newcommand{\alt}{~~|~~}
\newcommand{\comp}[1]{\llbracket #1 \rrbracket}

\newcommand{\inlinexp}[1]{
{\footnotesize
 \[\begin{array}{l}
 #1
 \end{array}\]}}

\newcommand{\inlinexpa}[2]{
{\footnotesize
 \[\begin{array}{#1}
 #2
 \end{array}\]}}
{% endhighlight %}

These helpers are meant primarily for layout and ease of writing later:
the alternatives definition will allow us to properly lay out our semantics,
while the compiler operator is just a nice extra. The `\inlinexp` and
`\inlinexpa` definitions are for placing expressions in the middle of prose,
which we'll get to later.

With these forms defined, we turn to our language forms, first in `defs.tex`
and then as a reference table in the main document. For this example, I will be
constructing a lambda calculus with a few extra terms.

{% highlight latex %}
%% FILENAME: defs.tex

%%% ...

%%------------------------------------------------------------------------
%% EXPRESSION MACROS
%%------------------------------------------------------------------------

%% lambda
\newcommand{\lamdefe}  [2] {\lambda #1.~#2}
\newcommand{\lamdefea} [2] {\begin{array}{l}\lambda#1.\\
                                            \hspace*{.5em}#2\\\end{array}}

%% let
\newcommand{\letdefe}    [3] {\letbind{#1}{#2}~\letin{#3}}
\newcommand{\letdefarre} [3] {\begin{array}{l}\letbind{#1}{#2}\\
                                              \letin{#3})\\\end{array}}

\newcommand{\letbind}  [2] {\mathsf{let}~\lbind{#1}{#2}}
\newcommand{\letbindp} [2] {\mathsf{let}~(\lbind{#1}{#2})}
\newcommand{\lbind}    [2] {#1=#2}
\newcommand{\letin}    [1] {\mathsf{in}~#1}

%% if
\newcommand{\ife}      [3] {\ifline{#1}~\thenline{#2}~\elseline{#3}}
\newcommand{\ifea}     [3] {\begin{array}{l}\ifline{#1}\\
                                            \thenline{#2}\\
                                            \elseline{#3}\end{array}}

\newcommand{\ifop}         {\mathsf{if}}
\newcommand{\ifline}   [1] {\ifop~ #1}
\newcommand{\thenline} [1] {\mathsf{then}~#1}
\newcommand{\elseline} [1] {\mathsf{else}~#1}

%% opers
\newcommand{\binopdef}     {\mathit{binop}}
\newcommand{\unopdef}      {\mathit{unop}}
\newcommand{\binope}   [2] {\binopdef~#1~#2}
\newcommand{\unope}    [1] {\unopdef~#1}

\newcommand{\andop}        {\mathsf{and}}
\newcommand{\orop}         {\mathsf{or}}
\newcommand{\notop}        {\mathsf{not}}
\newcommand{\ande}     [2] {\mathsf{and}~#1~#2}
\newcommand{\ore}      [2] {\mathsf{or}~#1~#2}
\newcommand{\note}     [1] {\mathsf{not}~#1}

%% values
\newcommand{\falsev}     {\mathsf{false}}
\newcommand{\truev}      {\mathsf{true}}
{% endhighlight %}

A few things may stick out immediately:

1. **There are multiple definition forms for if, let, and lambda.** This is
   intentional: sometimes I want a lambda laid out in a single line, but during
   writing a paper it may be convenient to add a line break. This can get
   truly messy if done inside of an existing array environment, and the
   guesswork for spacing can be a nightmare. As such, you need only do it once
   and then enclose it in a macro (array environments nest well).
2. **The keyword operators are all extracted into their own macros.** Since we
   are sometimes using multiple possible language definition forms for typesetting,
   it is important to abstract out the keywords. Then if we want to change the
   syntax of `if` from Haskell-style `if E then E else E` to Scheme's 
   `if E E E`, we need only modify the `\thenline` and `\elseline` commands to
   have our changes propagate.
3. **The names all end with funny letters.** Since LaTeX lacks a type system,
   I find it helpful to keep track of what operators are meant to belong to which
   part of my language. To this end, expressions end with `e` (or `ea` for
   array-based layouts), values end with `v`, and operators all end with `op`. 
   Pick a naming suffix or prefix and stick with it. (I prefer the suffix for
   later readability.)

With these forms in mind, we now turn to constructing our first piece of LaTex:
the language definition. We'll do this with an array environment.

{% highlight latex %}
%% FILENAME: paper.tex

%%% ...

\begin{document}

\section{Language Definitions}

\[
  \begin{array}{rcl}
  e  &:=& x \alt v \alt e~e \alt \letdefe{x}{e}{e}\\
     &\alt& \ife{e}{e}{e} \alt \unop{e} \alt \binop{e}{e}\\
  \\ %% A line break before the next definition sets
  v  &:=& \lamdefe{x}{e} \alt \truev \alt \falsev\\
  \\
  \binopdef &:=& \andop \alt \orop\\
  \unopdef  &:=& \notop \\

  \end{array}
\]

\end{document}
{% endhighlight %}

I'm assuming you're at least a little familiar with 
[math environments and arrays](https://en.wikibooks.org/wiki/LaTeX/Mathematics) 
[[2](http://latex.wikia.com/wiki/List_of_LaTeX_environments#Math_environments),
 [3](http://latex.wikia.com/wiki/Array_(LaTeX_environment))]. If you're not, 
the quick explanation is that `\[` and `\]` enclose a math environment and the
array environment has three columns which are right-align, center-aligned, and
left-aligned respectively.

The important draw here, though, is the language forms: they simply use our
definitions. We have, in essence, used the LaTeX macro system to build a sort of
"domain-specific language" that allows us to (more or less) write the language
we are describing. Furthermore, consistently using these definitions means that
changing the syntax is as easy as changing the definition and rebuilding the
file.

As an aside, the application rule should seem a bit dissatisfying if you have
been following along: until now, I have insisted that every piece I build be
modular and abstracted, but I reduce application to `~`. This is, for me, a
matter of convenience: I most often typeset languages that use juxtaposition
for application, and so do not often define an application operation. If I were
working with an OO language or the like, I would define 
`\newcommand{\invoke} [3] {#1.#2(#3)}` and the like.

**Building Semantic Definitions and Reductions, Part 1**

With language definitions in place, we turn our focus to semantic reductions.
For this purpose I am going to construct a call-by-value reduction model, first
with denotational semantics and then with contexts and a reduction relation.

For the denotational approach, we will use inference-style judgements. To do this,
we will first construct a reduction relation and a small layout helper in the
definition file:

{% highlight latex %}
%% FILENAME: defs.tex

%%% ...

%%------------------------------------------------------------------------
%% DEFINITION HELPERS
%%------------------------------------------------------------------------

%%% ....

\newcommand{\infr}  [3] [] {\infer[\textsc{#1}]{#3}{#2}}
\newcommand{\iand}        {\qquad}

%%------------------------------------------------------------------------
%% REDUCTION RELATION MACROS
%%------------------------------------------------------------------------

\newcommand{\subst} [3] {#3 [#2 / #1]}
\newcommand{\dstep} [2] {#1 ~\Downarrow~ #2}
{% endhighlight %}

Now we're ready to write the full set of denotational rules, using the
substitution command for substitution and `\dstep` for denotational reductions.

For the reduction rules, we have two layout options:

1. We can use a `gather*` environment to allow LaTeX to flow our definitions.
2. We can use an `array` environment to precisely control how the rules get laid
   out.

I'm partial to the latter for papers where space is a problem because it allows
us to strictly control the space our rules take up. That said, I am going to
demonstrate `gather*` when writing the denotational semantics to give you a feel
for each. (The `gather` environment lacking the asterisk numbers the lines,
which I find distasteful.) Here is a definition of our language semantics:

{% highlight latex %}
%% FILENAME: paper.tex

%%% ....

\section{Semantics One}

\begin{gather*}
\infr[App]
  {\dstep{e_1}{\lamdefe{x}{e'}} \iand \dstep{e_2}{v} \iand \cdots}
  {\dstep{e_1~e_2}{\subst{x}{v}{e'}}}
~
\infr
  {\dstep{e_1}{v} \iand \cdots}
  {\dstep{\letdefe{x}{e_1}{e_2}}{\subst{x}{v}{e_2}}}
\\
\infr
  {\dstep{e_1}{\truev} \iand \dstep{e_2}{v}}
  {\dstep{\ife{e_1}{e_2}{e_3}}{v}}
~
\infr
  {\dstep{e_1}{\falsev} \iand \dstep{e_3}{v}}
  {\dstep{\ife{e_1}{e_2}{e_3}}{v}}
\\
\infr
  {\dstep{e_1}{\truev} \iand \dstep{e_2}{\truev}}
  {\dstep{\ande{e_1}{e_2}}{\truev}}
~
\infr
  {\dstep{e_1}{\falsev}}
  {\dstep{\ande{e_1}{e_2}}{\falsev}}
\\
\infr
  {\dstep{e_1}{\truev}}
  {\dstep{\ore{e_1}{e_2}}{\truev}}
~
\infr
  {\dstep{e_1}{\falsev} \iand \dstep{e_2}{v}}
  {\dstep{\ore{e_1}{e_2}}{v}}
\\
\infr
  {\dstep{e}{\truev}}
  {\dstep{\note{e}}{\falsev}}
~
\infr
  {\dstep{e}{\falsev}}     
  {\dstep{\note{e}}{\truev}}
\end{gather*}
{% endhighlight %}

There are several pieces I should mention here:

1. The `\iand` command serves as a spacer for premises in judgments.
2. The `\cdots` in the premises of the application and let are meant to
   represent the substitution constraints that I have left out.
4. Rules may be named, as in the first rule, by passing an additional, optional argument.
5. In a `gather*` environment, rules must have spaces inserted between them. I use
   `~` between rules on the same line and `\\` for line breaks. This ends up a bit
   cramped in my opinion, and if I were to stick with this approach I'd likely 
   define `\newcommand{\rulespacer}{\qquad}` and use that in place of the `~`.

**Building Semantic Definitions and Reductions, Part 2**

For the small-step operational semantics, we will, as promised, use an array
environment. While I have, more recently, taken to also laying out SSOS using
judgment rules, for the sake of demonstration I will use a more traditional layout
approach. 

Before we begin, we must, of course, define a new stepping relation for each of
the redex and the context step. While I'm at it, I will also define a form for
easy layout of each line. While we're at it, we will also define language
evaluation contexts:

{% highlight latex %}
%% FILENAME: defs.tex

%%% ...

%%------------------------------------------------------------------------
%% DEFINITION HELPERS
%%------------------------------------------------------------------------

%%% ...

\newcommand{\Ctxt}       {\mathcal{E}}
\newcommand{\InCtxt} [1] {\Ctxt[#1]}

%%------------------------------------------------------------------------
%% REDUCTION RELATION MACROS
%%------------------------------------------------------------------------

%%% ....

\newcommand{\ssosredex}        {\rightarrow}
\newcommand{\ctxtreduce}       {\mapsto}
\newcommand{\sstep}     [3] [] {#2 &\ssosredex&  #3 &\textsc{#1}}
\newcommand{\ctxtstep}  [3] [] {#2 &\ctxtreduce& #3 &\textsc{#1}}
{% endhighlight %}

Next, we introduce evaluation contexts to our language definitions:

{% highlight latex %}
%% FILENAME: paper.tex

%%% ...

\begin{document}

\section{Language Definitions}

\[
  \begin{array}{rcl}
  %%% ...
  \\
  \Ctxt &:=& \Ctxt~e \alt v~\Ctxt \alt \letdefe{x}{\Ctxt}{e}\\
        &\alt& \ife{\Ctxt}{e}{e} \alt \unop{\Ctxt} %% no array break 
         \alt  \binop{\Ctxt}{e} \alt \binop{v}{\Ctxt}\\
  \end{array}
\]
{% endhighlight %}

Now we'll introduce the small-step semantics with our array environment,
defining both the redexes and the context reduction:

{% highlight latex %}
%% FILENAME: paper.tex

%%% ...

\section{Semantics Two}

\[
  \begin{array}{rcll}
  \sstep  {(\lamdefe{x}{e})~v}      {\subst{x}{v}{e} (\cdots)}{App}\\
  \sstep  {\letdefe{x}{v}{e}}       {\subst{x}{v}{e} (\cdots)}\\
  \sstep  {\ife{\truev}{e_2}{e_3}}  {e_2}\\
  \sstep  {\ife{\falsev}{e_2}{e_3}} {e_3}\\
  \sstep  {\ande{\falsev}{e_2}}     {\falsev}\\
  \sstep  {\ande{\truev}{e_2}}      {e_2}\\
  \sstep  {\ore{\falsev}{e_2}}      {e_2}\\
  \sstep  {\ore{\truev}{e_2}}       {\truev}\\
  \sstep  {\note{\falsev}}          {\truev}\\
  \sstep  {\note{\truev}}           {\falsev}\\
  \\
  \ctxtstep {\InCtxt{e}}            {\InCtxt{e'} ~ (\text{if } e\ssosredex e')}\\
  \end{array}
\]
{% endhighlight %}

**Constructing Typing Judgements**

The last thing I will do is lay out the typing rules for our little calculus. Since
booleans are our only base type, this should be fairly easy. As before, we first
introduce a series of definitions to our definition file. While I'm at it, I will
also define the type environment and judgment relation for types.

{% highlight latex %}
%% FILENAME: defs.tex

%%% ...

%%------------------------------------------------------------------------
%% TYPE DEFINITION MACROS
%%------------------------------------------------------------------------

\newcommand{\funct} [2] {#1\nobreak\rightarrow\nobreak#2}
\newcommand{\boolt}     {\mathtt{bool}}

\newcommand{\typeEnv}         {\Gamma}
\newcommand{\entails}         {\vdash}
\newcommand{\judgment}   [3] {#1 \entails #2 : #3}
\newcommand{\envent}      [2] {\judgment{\typeEnv}{#1}{#2}}
\newcommand{\extenvent}   [4] {\judgment{\typeEnv, #1 : #2}{#3}{#4}}
\newcommand{\envlookup}   [3] {\infr{#1(#2) = #3}{\judgment{#1}{#2}{#3}}}
{% endhighlight %}

The macros we will use are the last three defined: 

- `envent`, short for 'environment entails'
- `extenvent`,  short for 'extended environment entails'
- `envlookup`, for auto-generating  environment lookups

Now we extend the definitions in our main file and scribble
out the typing judgments. I will again use an array environment, this
time demonstrating how one can be used to lay out inference rules.

{% highlight latex %}
%% FILENAME: paper.tex

%%% ...

\begin{document}

\section{Language Definitions}

\[
  \begin{array}{rcl}
  %%% ...
  \\
  \tau  &:=& \boolt \alt \funct{\tau}{\tau}\\
  \end{array}
\]

%%% ...

\section{Typing Judgments}

\[
  \begin{array}{cc}
    \infr{}{\envent{\truev}{\boolt}}
    &
    \infr{}{\envent{\falsev}{\boolt}}
    \\\\
    \envlookup{\typeEnv}{x}{\tau} 
    &
    \infr
      {\extenvent{x}{\tau}{e}{\tau'}}
      {\envent{\lamdefe{x}{e}}{\funct{\tau}{\tau'}}} 
    \\\\
    \infr
      {\envent{e_1}{\funct{\tau}{\tau'}} \iand 
       \envent{e_2}{\tau}}
      {\envent{e_1~e_2}{\tau'}} 
    &
    \infr
      {\extenvent{x}{\tau'}{e_2}{\tau} \iand
       \envent{e_1}{\tau'}}
      {\envent{\letdefe{x}{e_1}{e_2}}{\tau}}
    \\\\
      \infr
        {\envent{e_1}{\boolt} \iand \envent{e_2}{\tau} \iand \envent{e_3}{\tau}}
        {\envent{\ife{e_1}{e_2}{e_3}}{\tau}}
    &
    \infr
      {\envent{\unopdef}{\funct{\tau'}{\tau}} \iand
       \envent{e}{\tau'}}
      {\envent{\unope{e}}{\tau}}
    \\\\
    \multicolumn{2}{c}{ %% for a really LOOOOONG rule
      \infr
        {\envent{\binopdef}{\funct{\tau_1}{\funct{\tau_2}{\tau}}} \iand
         \envent{e_1}{\tau_1}                                     \iand
         \envent{e_2}{\tau_2}}
        {\envent{\binope{e_1}{e_2}}{\tau}}
    }
    \\\\
  \end{array}
\]
{% endhighlight %}

**Conclusion**

I hope you have found this little tutorial insightful and helpful. Again, the
code for all of this is [available on GitHub](https://github.com/cgswords/texsem).

If I've made any mistake in the presentation here versus the LaTeX code, or in
the semantic definitions I've put forth, please let me know. 

In closing, here are some parting comments:

1. If you'd like to write SSOS using the judgment approach, simply write
   something like `\dstep` and use it.
2. If you need an additional environment (e.g., for channels) in your
   reduction relation, I find it convenient to build two versions: one with
   the default symbol and one that takes the symbol as input, as with the
   rules for type environment extension above.
3. The [xspace](https://www.ctan.org/pkg/xspace] package is invaluable for
   using commands and keywords within prose.
4. All of the commands above, properly, should use `\ensuremath`. I'm too lazy
   to type that everywhere, so I don't, but if you are going to share such a
   definition set with other people you may consider it.
5. You may consider the [stmaryd](https://www.ctan.org/pkg/stmaryrd?lang=en)
   package for additional symbols (as needed).
6. A lot of people seem to like [Ben C Pierce's Rules](http://www.cis.upenn.edu/~bcpierce/papers/bcprules.sty)
  for typing judgments. They don't require a math environment.
