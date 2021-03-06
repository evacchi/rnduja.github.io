---
layout:     post
title:      "A tour of Factor: 3"
subtitle:   "Modules and testing"
date:       2016-06-20 12:00:00
author:     "Andrea Ferretti"
header-img: "img/chain.jpg"
comments:    true
tags:       [concatenative-programming]
---


## Vocabularies

It is now time to start writing your functions in files and learn how to import
them in the listener. Factor organizes words into nested namespaces called
**vocabularies**. You can import all names from a vocabulary with the word `USE:`.
In fact, you may have seen something like

```factor
USE: math.ranges
```

when you asked the listener to import the word `[1,b]` for you. You can also use
more than one vocabulary at a time with the word `USING:`, which is followed by
a list of vocabularies and terminated by `;`, like

```factor
USING: math.ranges sequences.deep ;
```

Finally, you define the vocabulary where your definitions are stored with the
word `IN:`. If you search the online help for a word you have defined so far,
like `prime?`, you will see that your definitions have been grouped under the
default `scratchpad` vocabulary. By the way, this shows that the online help
automatically collects information about your own words, which is a very useful
feature.

There are a few more words, like `QUALIFIED:`, `FROM:`, `EXCLUDE:` and `RENAME:`,
that allow more fine-grained control over the imports, but `USING:` is the most
common.

On disk, vocabularies are stored under a few root directories, much like with the
classpath in JVM languages. By default, the system starts looking up into the
directories `basis`, `core`, `extra`, `work` under the Factor home. You can add
more, both at runtime with the word `add-vocab-root`, and by creating a
configuration file `.factor-rc`, but for now we will store our vocabularies
under the `work` directory, which is reserved for the user.

Generate a template for a vocabulary writing

```factor
USE: tools.scaffold
"github.tutorial" scaffold-work
```

You will find a file `work/github/tutorial/tutorial.factor` containing an empty
vocabulary. Factor integrates with many editors, so you can try
`"github.tutorial" edit`: this will prompt you to choose your favourite editor,
and use that editor to open the newly created vocabulary.

You can add the definitions of the previous paragraph, so that it looks like

```factor
! Copyright (C) 2016 Andrea Ferretti.
! See http://factorcode.org/license.txt for BSD license.
USING: ;
IN: github.tutorial

: [2,b] ( n -- {2,...,n} ) 2 swap [a,b] ; inline

: multiple? ( a b -- ? ) swap divisor? ; inline

: prime? ( n -- ? ) [ sqrt [2,b] ] [ [ multiple? ] curry ] bi any? not ;
```

Since the vocabulary was already loaded when you scaffolded it, we need a way
to refresh it from disk. You can do this with `"github.tutorial" refresh`.
There is also a `refresh-all` word, with a shortcut `F2`.

You will be prompted a few times to use vocabularies, since your `USING:`
statement is empty. After having accepted all of them, Factor suggests you a
new header with all the needed imports:

```factor
USING: kernel math.functions math.ranges sequences ;
IN: github.tutorial
```

Now that you have some words in your vocabulary, you can edit, say, the
`multiple?` word with `\ multiple? edit`. You will find your editor open on the
relevant line of the right file. This also works for words in the Factor
distribution, although it may be a bad idea to modify them.

This `\` word requires a little explanation. It works like a sort of escape,
allowing us to put a reference to the next word on the stack, without executing
it. This is exactly what we need, because `edit` is a word that takes words
themselves as arguments. This mechanism is similar to quotations, but while a
quotation creates a new anonymous function, here we are directly refering to
the word `multiple?`.

Back to our task, you may notice that the words `[2,b]` and `multiple?` are
just helper functions that you may not want to expose directly. To hide them
from view, you can wrap them in a private block like this

```factor
<PRIVATE

: [2,b] ( n -- {2,...,n} ) 2 swap [a,b] ; inline

: multiple? ( a b -- ? ) swap divisor? ; inline

PRIVATE>
```

After making this change and refreshed the vocabulary, you will see that the
listener is not able to refer to words like `[2,b]` anymore. The `<PRIVATE` word
works by putting all definitions in the private block under a different
vocabulary, in our case `github.tutorial.private`.

It is still possible to refer to words in private vocabularies, as you can
confirm by searching for `[2,b]` in the online help, but of course this is
discouraged, since people do not guarantee any API stability for private words.
Words under `github.tutorial` can refer to words in `github.tutorial.private`
directly, like `prime?` does.

## Tests and documentation

This is a good time to start writing some unit tests. You can create a skeleton
with

```factor
"github.tutorial" scaffold-tests
```

You fill find a generated file under `work/github/tutorial/tutorial-tests.factor`,
that you can open with `"github.tutorial" edit-tests`. Notice the line

```factor
USING: tools.test github.tutorial ;
```

that imports the unit testing module as well as your own. We will only test the
public `prime?` function.

Tests are written using the `unit-test` word, which expects two quotations:
the first one containing the expected outputs and the second one containing
the words to run in order to get that output. Add these lines to
`github.tutorial-tests`:

```factor
[ t ] [ 2 prime? ] unit-test
[ t ] [ 13 prime? ] unit-test
[ t ] [ 29 prime? ] unit-test
[ f ] [ 15 prime? ] unit-test
[ f ] [ 377 prime? ] unit-test
[ f ] [ 1 prime? ] unit-test
[ t ] [ 20750750228539 prime? ] unit-test
```

You can now run the tests with `"github.tutorial" test`. You will see that we
have actually made a mistake, and pressing `F3` will show more details. It seems
that our assertions fails for `2`.

In fact, if you manually try to run our functions for `2`, you will see that our
defition of `[2,b]` returns `{ 2 }` for `2 sqrt`, due to the fact that the
square root of two is less than two, so we get a descending interval.
Try making a fix so that the tests now pass.

There are a few more words to test errors and inference of stack effects.
`unit-test` suffices for now, but later on you may want to check `must-fail`
and `must-infer`.

We can also add some documentation to our vocabulary. Autogenerated
documentation is always available for user-defined words (even in the listener),
but we can write some useful comments manually, or even add custom articles
that will appear in the online help. Predictably, we start with
`"github.tutorial" scaffold-docs` and then `"github.tutorial" edit-docs`.

The generated file `work/github/tutorial-docs.factor` imports `help.markup` and
`help.syntax`. These two vocabularies define words to generate documentation.
The actual help page is generated by the `HELP:` parsing word.

The arguments to `HELP:` are nested array of the form `{ $directive content... }`.
In particular, you see here the directives `$values` and`$description`, but a
few more exist, such as `$errors`, `$examples` and `$see-also`.

Notice that the type of the output `?` has been inferred to be boolean. Change
the first lines to look like

```factor
USING: help.markup help.syntax kernel math ;
IN: github.tutorial

HELP: prime?
{ $values
    { "n" fixnum }
    { "?" boolean }
}
{ $description "Tests if n is prime. n is assumed to be a positive integer." } ;
```

and refresh the `github.tutorial` vocabulary. If you now look at the help for
`prime?`, for instance with `\ prime? help`, you will see the updated documentation.

You can also render the directives in the listener for quicker feedback. For
instance, try writing

```factor
{ $values
    { "n" integer }
    { "?" boolean }
} print-content
```

The help markup contains a lot of possible directives, and you can use them to
write stand-alone articles in the help system. Have a look at some more with
`"element-types" help`.

In the next post, we will have a look at the object-oriented features of
Factor.

Until then!