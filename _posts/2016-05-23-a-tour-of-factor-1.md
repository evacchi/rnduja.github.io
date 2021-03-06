---
layout:     post
title:      "A tour of Factor: 1"
subtitle:   "Getting your feet wet with concatenative programming"
date:       2016-05-23 12:00:00
author:     "Andrea Ferretti"
header-img: "img/chain.jpg"
comments:    true
tags:       [concatenative-programming]
---

# A panoramic tour of Factor

This series is a repost from [A panoramic tour of Factor](http://andreaferretti.github.io/factor-tutorial/).
I will split the original tutorial in more manageable chunks, but the content
will stay mostly the same.

[Factor](http://factorcode.org) is a mature, dynamically typed language based on
the concatenative paradigm. Getting started with Factor can be daunting since
the concatenative paradigm is different from most mainstream languages. This
tutorial will guide you through the basics of Factor so you can appreciate its
simplicity and power.

I assume you are an experienced programmer familiar with a functional language,
and I'll assume you understand concepts like
[folding](http://en.wikipedia.org/wiki/Fold_%28higher-order_function%29),
[higher-order functions](http://en.wikipedia.org/wiki/Higher-order_function),
and [currying](http://en.wikipedia.org/wiki/Currying).

Even though Factor is a niche language, it is mature and has a comprehensive
standard library covering tasks from JSON serialization to socket programming
and HTML templating. It runs in its own optimized VM with very high performance
for a dynamically typed language. It also has a flexible object system, a
[FFI](http://en.wikipedia.org/wiki/Foreign_function_interface) to C, and
asynchronous I/O that works a bit like Node.js, but with a much simpler model
for cooperative multithreading.

You may wonder why you should care enough about Factor to read this tutorial.
Factor has a few significant advantages over other languages, most arising from
the fact that it has essentially no syntax:

* refactoring is very easy, leading to short and meaningful function definitions;
* it is extremely succinct, letting the programmer concentrate on what is important instead of boilerplate;
* it has powerful metaprogramming capabilities, exceeding even those of LISPs;
* it is ideal to create [DSLs](http://en.wikipedia.org/wiki/Domain-specific_language);
* it integrates easily with powerful tools.

Before you start this tutorial, [download a copy of Factor](http://factorcode.org)
so you can follow along with the examples in the listener
(the Factor [REPL](http://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop)).

I assume you are using Mac OS X or some distribution of Linux, but everything
should work the same on other systems, provided you adjust the file paths in
the examples.

The first section gives some motivation for the rather peculiar model of
computation of concatenative languages, but feel free to skip it if you want to
get your feet wet and return to it after some hands on practice with Factor.

## Concatenative languages

Factor is a *concatenative* programming language in the spirit of
[Forth](http://en.wikipedia.org/wiki/Forth_%28programming_language%29).
What does this even mean?

To understand concatenative programming, imagine a world where every value is a
function, and the only operation allowed is function composition. Since function
composition is so pervasive, it is implicit, and functions can be literally
juxtaposed in order to compose them. So if `f` and `g` are two functions, their
composition is just `f g` (unlike in mathematical notation, functions are read
from left to right, so this means first execute `f`, then execute `g`).

This requires some explanation, since we know functions often have multiple
inputs and outputs, and it is not always the case that the output of `f` matches
the input of `g`. For instance, `g` may need access to values computed by earlier
functions. But the only thing that `g` can see is the output of `f`, so the
output of `f` is the whole state of the world as far as `g` is concerned.
To make this work, functions have to thread the global state, passing it to each
other.

There are various ways this global state can be encoded. The most naive would
use a hashmap that maps variable names to their values. This turns out to be too
flexible: if every function can access any piece of global state, there is
little control on what functions can do, little encapsulation, and ultimately
programs become an unstructured mess of routines mutating global variables.

It works well in practice to represent the state of the world as a stack.
Functions can only refer to the topmost element of the stack, so that elements
below it are effectively out of scope. If a few primitives are given to
manipulate a few elements on the stack (e.g., `swap`, that exchanges the top
two elements on the stack), then it becomes possible to refer to values down
the stack, but the farther the value is down the stack, the harder it becomes
to refer to it.

So, functions are encouraged to stay small and only refer to the top two or
three elements on the stack. In a sense, there is no distinction between local
and global variables, but values can be more or less local depending on their
distance from the top of the stack.

Notice that if every function takes the state of the whole world and returns
the next state, its input is never used anymore. So, even though it is convenient
to think of pure functions as receiving a stack as input and outputting a stack,
the semantics of the language can be implemented more efficiently by mutating a
single stack.

This leaves concatenative languages like Factor in a strange position: they are
both extremely functional - only allowing composition of simpler functions into
more complex ones - and largely imperative - describing operations on a mutable
stack.

## Playing with the stack

Let us start looking what Factor actually feels like. Our first words will be
literals, like `3`, `12.58` or `"Chuck Norris"`. Literals can be thought as
functions that push themselves on the stack. Try writing `5` in the listener and
then press enter to confirm. You will see that the stack, initially empty, now
looks like

    5

You can enter more that one number, separated by spaces, like `7 3 1`, and get

    5
    7
    3
    1

(the interface shows the top of the stack on the bottom). What about operations?
If you write `+`, you will run the `+` function, which pops the two topmost
elements and pushes their sum, leaving us with

    5
    7
    4

You can put additional inputs in a single line, so for instance `- *` will leave
the single number `15` on the stack (do you see why?).

The function `.` (a period or a dot) prints the item at the top of the stack,
while popping it out of the stack, leaving the stack empty.

If we write everything on one line, our program so far looks like

    5 7 3 1 + - * .

which shows Factor's peculiar way of doing arithmetic by putting the arguments
first and the operator last - a convention which is called
[Reverse Polish Notation](https://en.wikipedia.org/wiki/Reverse_Polish_notation)
(RPN). Notice that RPN requires no parenthesis, unlike the
[polish notation](http://en.wikipedia.org/wiki/Polish_notation) of Lisps where
the operator comes first, and RPN requires no precedence rules, unlike the
[infix notation](http://en.wikipedia.org/wiki/Infix_notation) used in most
programming languages and in everyday arithmetic. For instance in any Lisp,
the same computation would be written

    (* 5 (- 7 (+ 3 1)))

and in familiar infix notation

    (7 - (3 + 1)) * 5

Also notice that we have been able to split our computation onto many lines or
combine it onto fewer lines rather arbitrarily, and that each line made sense
in itself.

## Defining our first word

We will now define our first function. Factor has slightly odd naming of
functions: since functions are read from left to right, they are simply called
**words**, and this is what we'll call them from now on. Modules in Factor
define words in terms of previous words and these sets of words are then
called **vocabularies**.

Suppose we want to compute the [factorial](http://en.wikipedia.org/wiki/Factorial).
To start with a concrete example, we'll compute the factorial of `10`, so we
start by writing `10` on the stack. Now, the factorial is the product of the
numbers from `1` to `10`, so we should produce such a list of numbers first.

The word to produce a range is called `[a,b]` (tokenization is trivial in Factor
because words are always separated by spaces, so this allows you to use any
combination of non-whitespace characters as the name of a word; there are no
semantics to the `[`, the `,` and the `]` in `[a,b]` since it is just a token
like `foo` or `bar`).

The range we want starts with `1`, so we can use the simpler word `[1,b]` that
assumes the range starts at `1` and only expects the value at the top of the
range to be on the stack. If you write `[1,b]` in the listener, Factor will
prompt you with a choice, because the word `[1,b]` is not imported by default.
Factor is able to suggest you import the `math.ranges` vocabulary, so choose
that option and proceed.

You should now have on your stack a rather opaque structure which looks like

```factor
T{ range f 1 10 1 }
```

This is because our range functions are lazy and only create the range when we
attempt to use it. To confirm that we actually created the list of numbers
from `1` to `10`, we convert the lazy response on the stack into an array
using the word `>array`. Enter that word and your stack should now look like

```factor
{ 1 2 3 4 5 6 7 8 9 10 }
```

which is promising!

Next, we want to take the product of those numbers. In many functional languages,
this could be done with a function called reduce or fold. Let's look for one.
Pressing `F1` in the listener will open a contextual help system, where you can
search for `reduce`. It turns out that `reduce` is actually the word we are
looking for, but at this point it may not be obvious how to use it.

Try writing `1 [ * ] reduce` and look at the output: it is indeed the factorial
of `10`. Now, `reduce` usually takes three arguments: a sequence (and we had one
on the stack), a starting value (this is the `1` we put on the stack next) and
a binary operation. This must certainly be the `*`, but what about those square
brackets around the `*`?

If we had written just `*`, Factor would have tried to apply multiplication to
the topmost two elements on the stack, which is not what we wanted. What we need
is a way to get a word onto the stack without applying it. Keeping to our
textual metaphor, this mechanism is called **quotation**. To quote one or more
words, you just surround them by `[` and `]` (leaving spaces!). What you get is
akin to an anonymous function in other languages.

Let's type the word `drop` into the listener to empty the stack, and try
writing what we have done so far in a single line: `10 [1,b] 1 [ * ] reduce`.
This will leave `3628800` on the stack as expected.

We now want to define a word for factorial that can be used whenever we want a
factorial. We will call our word `fact` (although `!` is customarily used as the
symbol for factorial, in Factor `!` is the word used for comments). To define it,
we first need to use the word `:`. Then we put the name of the word being defined,
then the **stack effects** and finally the body, ending with the `;` word:

```factor
: fact ( n -- n! ) [1,b] 1 [ * ] reduce ;
```

What are stack effects? In our case it is the `( n -- n! )`. Stack effects are
how you document the inputs from the stack and outputs to the stack for your word.
You can use any identifier to name the stack elements - here we use `n`. Factor
will perform a consistency check that the number of inputs and outputs you
specify agrees with what the body does.

If you try to write

```factor
: fact ( m n -- n! ) [1,b] 1 [ * ] reduce ;
```

Factor will signal an error that the 2 inputs (`m` and `n`) are not consistent
with the body of the word. To restore the previous correct definition press
`Ctrl+P` two times to get back to the previous input and then enter it.

We can think at the stack effects in definitions both as a documentation tool
and as a very simple type system, which nevertheless does catch a few errors.

In any case, you have succesfully defined your first word: if you write `10 fact`
in the listener you can prove it.

Notice that the `1 [ * ] reduce` part of the definition sort of makes sense on
its own, being the product of a sequence. The nice thing about a concatenative
language is that we can just factor this part out and write

```factor
: prod ( {x1,...,xn} -- x1*...*xn ) 1 [ * ] reduce ;
: fact ( n -- n! ) [1,b] prod ;
```

Our definitions have become simpler and there was no need to pass parameters,
rename local variables, or do anything else that would have been necessary to
refactor our function in most languages.

Of course, Factor already has a word for the factorial (actually there is a
whole `math.factorials` vocabulary, including many variants of the usual
factorial) and a word for the product (`product` in the `sequences` vocabulary),
but as it often happens introductory examples overlap with the standard library.

In the next post we will start to look at some basic words called **combinators**
that are used as fundamental building blocks for more complex operations.

Until then!
