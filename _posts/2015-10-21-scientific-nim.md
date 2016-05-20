---
layout:     post
title:      "Nim for scientific computing"
subtitle:   "Back to the future"
date:       2015-10-21 12:00:00
author:     "Andrea Ferretti"
header-img: "img/2015-10-21-scientific-nim/cover.jpg"
comments:   true
tags:       [nim,data-science]
---


What is Nim and why it matters for scientific computing
=======================================================

In the last few months, I have been shifting the focus of my work
towards scientific computing, be it for cryptographic applications,
machine learning or neural networks. I have been hard-pressed to
find an environment that satisfies me fully.

I do most of my daily work in Scala, and while I am still a big fan
of it, trying to make it into a tool for scientific programming
often hits its limits. A few things are desirable:

* raw speed, in particular when writing inner loops
* a predictable memory layout is necessary to be cache-friendly and
  to interface easily with C libraries
* the possibility to interface easily with dedicated hardware such
  as GPUs and FPGAs
* did I mention speed?

To give an example of the first problem, a nested loop in Scala
typically looks like

~~~scala
for {
  xs <- xss
  x <- xs
} yield x * x
~~~

which the first steps of `scalac` transform into

~~~scala
xss flatMap { xs =>
  xs map { x =>
    x *x
  }
}
~~~

This adds the overhead of calling a function inside each loop step,
and while this will be inlined by the JVM JIT, it would be nice to
be able to ensure that this happens all the time without worrying
about whether the runtime is doing what we wanted to do in the first
place.

There are a lot of projects - both for the JVM and specific for Scala -
that try to do one of the following:

* remove function calls overhead
* remove overhead and simplify the interaction with native libraries
* allow a predictable memory layout (for instance, arrays of contiguous objects)
* decrease the pressure on the GC, by avoiding heap memory allocations

[I](http://openjdk.java.net/jeps/169)
[will](https://github.com/tiarkrompf/scala-virtualized/wiki)
[just](https://github.com/densh/scala-offheap)
[mention](https://github.com/xerial/larray)
[a](http://scala-miniboxing.org/)
[few](http://www.mapdb.org/benchmarks.html)
[of](http://trove.starlight-systems.com/)
[these](https://github.com/bytedeco/javacpp)
[efforts](http://openjdk.java.net/projects/panama/)

Eventually, the JVM will get value types and other goodies that simplify the
interaction with native libraries, or Scala will get a native backend. But in
the meantime, I would rather look elsewhere when I need to do some heavy
computations.

Outside the JVM
---------------

Since my aim was to interface with C libraries, I started looking
outside the JVM, to avoid the nastiness and overhead of JNI and
related interfaces. A few options pop out.

A lot of scientific work is still done in C or C++, as well as in
Fortran. I was looking for something more high level that allows
a more rapid development. In particular, C is severely limited in
expressivity by the lack of parametric polymorphism, which in turn
renders essentially impossible to have good libraries of data
structures. C++ is a tempting option, especially with the additions
in C++11, but realistically most of C++ code is not written to
make use of the new features of the standard, and the historical
baggage of C++ brings a good deal of complexity.

Python is clearly the most used option right now, but it is also
lacking. While it probably features the most complete ecosystem
of libraries, I always found the way to interface with native code
disappointing.

In theory, one would write the heavy crunching numerics in a
compiled language (typically C) and then glue together these blocks
using Python. In practice, this often means that

* at each moment, one has to choose between speed and ease of use.
  It is difficult to write a complex algorithm (for which one would
  want to make use of Python) that **also** has to run fast;
* for each C library, one has to write glue that is Python-aware,
  in particular incrementing or decrementing reference counts where
  appropriated and taking and releasing the GIL. This is difficult
  and error-prone;
* the native extensions usually do not work under PyPy, which is
  otherwise the most convenient Python interpreter

Similarly, one could use Lua with [Torch](http://torch.ch/), and
while I am making use of it (as well as Python, in fact), it
suffers from the same weaknesses.

Also, a lot of neural network code is error-prone and rather
difficult to test - in fact a lot of functions are not used
directly, but rather put inside an optimizer, which makes even
more difficult to detect errors in complex network architectures -
and so, I would really like to get the support from types as a
guide.

Enter Nim
---------

[Nim](http://nim-lang.org/) is a statically typed language
which compiles to C (as well as JS), which makes interoperability
with C a breeze. It is mostly imperative, but has a lot of
features that allow a functional style of programming where
appropriate. I think that the feature that defines it mostly is
the reliance on compile time metaprogramming, in the LISP
tradition. Above all, Nim is very high-level, while still
mantaining complete control over memory, much as one would
expect from C.

Let me give a simple example of computing the average of a
sequence of points just to give an idea of the flavour:

~~~nimrod
import sequtils

type Point = tuple[x, y: float]

proc `+`(p, q: Point): Point = (p.x + q.x, p.y + q.y)

proc `/`(p: Point, k: float): Point = (p.x / k, p.y / k)

proc average(points: seq[Point]): Point =
  foldl(points, a + b) / float(points.len)
~~~

The use of `foldl` here makes it look like what one would
write in a functional language, but in fact it is an example
of a [Nim template](http://nim-lang.org/docs/manual.html#templates).
It makes use of metaprogramming to ensure that the expression `a + b`
is inlined at compile time, so that the result has zero overhead
with respect to an imperative loop.

Nim as enhanced C
-----------------

A first way to think about Nim is just as a better C. Most of Nim
features are designed so that they translate transparently to C,
hence you can predict rather well how Nim programs are compiled.

Apart from basic types such as `int64` or `float32`, Nim has complex
types in the form of object, defined like

~~~nimrod
type Circle = object
  x, y, radius: float
~~~

that map to the corresponding C structs. Also, Nim functions - declared
with `proc` - are compiled into C functions. Calling C is then rather
easy, since objects have the same layout in memory, and functions are the
same. Here we import the `malloc` function from the C standard library:

~~~nimrod
proc malloc(size: uint): pointer {.header: "<stdlib.h>", importc: "malloc".}
~~~

On top of this, Nim, adds a lot of features that make programming simpler
while adding no runtime overhead. The most prominent departure from the C
tradition is a leaner syntax inspired by Python. Another one is the fact
that Nim has a proper module system: everything is namespaced to modules,
of which you can import some or all members (types, functions, constants...).
This avoids the mess of textually concatenating headers and protecting them
with flags. The module system, coupled with the package manager
[Nimble](https://github.com/nim-lang/nimble), allows to handle dependencies
cleanly.

The other big deal with respect to C is the fact that Nim allows for polymorphic
functions, both generic and ad-hoc. Generic functions take type parameters and
are compiled by specialization to each concrete type for which they are called.
An example would be

~~~nimrod
proc last[A](xs: seq[A]): A = xs[xs.len - 1]
~~~

The fact that the types are checked only when a concrete type is provided allows
to write things such as

~~~nimrod
proc sum[A](xs: seq[A]): A = foldl(xs, a + b)
~~~

Notice that this only makes sense when `A` implements addition. Nim will check
this for a concrete type `A` when `sum` is actually called, that is, when `sum`
is specialized. This is like duck typing, but with the assurance that `sum` is
only ever called by ducks that actually quack!

One can also specify that `A` has to implement certain functions through the
mechanism of [concepts](http://nim-lang.org/docs/manual.html#generics-concepts)
but this is only seldom needed. This kind of duck typing for generics makes the
syntax very light (no interfaces to specify), just as in dynamic languages, but
with everything fully typed.

One also has ad-hoc polymorphism, that is, the possibility of defining a function
with the same name for different types. Nim will choose the implementation to
use based on type information. This is especially useful when interfacing with
C libraries that export many different implementations for the same type. For
instance in my [linear-algebra library](https://github.com/unicredit/linear-algebra)
I can import various BLAS routines under the same name, like

~~~nimrod
proc scal(N: int, ALPHA: float32, X: ptr float32, INCX: int)
  {. header: header, importc: "cblas_sscal" .}
proc scal(N: int, ALPHA: float64, X: ptr float64, INCX: int)
  {. header: header, importc: "cblas_dscal" .}
~~~

and be sure that the suitable implementation of `scal` will be called,
according to the arguments I pass.

Compile-time fanciness
----------------------

Up to this point, I hope I have shown how Nim can be used to replace C,
while allowing a few more high-level features. The real power of Nim comes
from the fact that it allows arbitrary computations at compile time, and
unlike C++, the compile time language is Nim itself instead of the template
sublanguage.

The first, rather trivial, example of this is the fact that constants
can be defined by calling functions, like

~~~nimrod
import math

const
  giga = 2 ^ 9
  seqNumber = random([1, 2, 3, 4, 5, 6])
~~~

For something more sophisticated, Nim has both [templates](http://nim-lang.org/docs/manual.html#templates)
and [macros](http://nim-lang.org/docs/manual.html#macros). Macros, like in
Lisp, allow arbitrary transformations on the AST, while templates are slightly
more limited and employ a declarative syntax.

Templates can be thought as functions that are always inlined, and as such
they share the function syntax. In fact, a way to ensure that `last` above
is inlined would be to just change the definition to

~~~nimrod
template last(xs: expr): expr = xs[xs.len - 1]
~~~

Macros, on the other hand, allow to introduce more complex language constructs.
A good example is my [Patty library](https://github.com/andreaferretti/patty)
for pattern matching. Nim, per se, does not have a concise way to define algebraic
data types, nor it has pattern matching facilities. With the macros defined in
Patty, one can write

~~~nimrod
variant Shape:
  Circle(r: float)
  Rectangle(w: float, h: float)
  UnitCircle

let r = Rectangle(10, 13)

match r:
  case Circle(_):
    echo "It is a circle"
  case Rectangle(w, h):
    echo "It is a rectangle of width ", $w
  case UnitCircle():
    echo "It is a UnitCircle"
~~~

The `match` above is transformed into a switch at compile time. Actually, the
support for pattern matching in Patty is quite limited (most prominently, there
is no support for nesting), but this is due to lack of time - I developed Patty
in my spare time - and there is no reason why one could not have support for
pattern matching on par with ML languages.

Nim also provides a form of [dependent types](https://en.wikipedia.org/wiki/Dependent_type),
in the form of [`static[T]`](http://nim-lang.org/docs/manual.html#special-types-static-t).
A value of type `T` known at compile time can be used as a type inside generics.
This allows to encode arbitrary constraints inside types.

An example of this is in my [linalg library](https://github.com/unicredit/linear-algebra),
where this feature is used to encode the dimensions of vectors and matrices
inside the type of the matrix itself. A typical usage of the library looks
like

~~~nimrod
import linalg

let
  v = randomVector(12)
  m = randomMatrix(8, 12)
  w = m * v
~~~

Here the type of `v` is for instance `Vector64[12]`, while `m` has type
`Matrix64[8, 12]`. As such, a usage like

~~~nimrod
echo m * randomVector(11)
~~~

does not compile, since dimensions do not match. Something similar can be done
to encode type-safe modular arithmetic, provided moduli are known at compile time.

At compile time, one also has the possibility to check arbitrary conditions. For
instance, the library `linalg` uses BLAS under the hood to perform matrix and
vector operations. One may want to provide operations for matrices over arbitrary
rings with a naive implementation, delegating to BLAS for the case of `float32`
or `float64`. To do this, one could just have type `Vector[N, A]` and `Matrix[M, N, A]`,
and then special-case operations for floats:

~~~nimrod
proc `*`[M, N, A](m: Matrix[M, N, A], v: Vector[N, A]): Vector[M, A] =
  when A is float32 or A is float64:
    # BLAS implementation
  else:
    # fallback implementation
~~~

Here the `when` keyword acts like `if` but is evaluated at compile time, opening
all range of checks on anything that is statically known. In fact, `when` is
also used in place of the common C preprocessor directives to distinguish
architectures or compile flags.

A nice application of this is in tandem with the `compiles` function, which takes
an arbitrary expression and return a boolean. This is used in the unit tests for
`linalg` to test that forbidden operations do not compile, as in

~~~nimrod
test "vector dimension should agree in a sum":
  let
    u = vector([1.0, 2.0, 3.0, 4.0, 5.0])
    v = vector([1.0, 2.0, 3.0, 4.0])
  when compiles(u + v): fail()
  when compiles(u - v): fail()
~~~

A final opportunity is to give user-defined optimizations in the form of
[rewrite rules on the AST](http://nim-lang.org/docs/manual.html#term-rewriting-macros).
These allow to pattern match on the AST and make your own optimizations.
I use them in `linalg` in order to rewrite an expression such as

~~~nimrod
echo v1 + 5.3 * v2
~~~

in terms of a single BLAS call (instead of a scalar multiplication followed by a sum).

Clean up after yourself
-----------------------

Up to this point, I have only described features that add no runtime overhead over
pure C. Turns out, Nim is actually [garbage collected](http://nim-lang.org/docs/gc.html).

This is not as bad for performance as it seems at first. The garbage collector uses
reference counting with cycle detection, and the compiler makes use of type information
to infer when cycles are impossible to form. One can also mark data structures as
acyclic explicitly, like this:

~~~nimrod
type
  Node = ref NodeObj
  NodeObj {.acyclic, final.} = object
    left, right: Node
    data: string
~~~

Different threads have each their own heap (there is also a shared heap for shared data),
so that heap scans are lighter. Moreover, the GC only fires during allocation, and it
is also possible to put hard limits on how long it runs, or disable it temporarily. All
of this gives a great control to guarantee predictable performance where needed.

Moreover, a lot of values can be allocated on the stack - in fact all the example I gave
in the previous sections did not make use of the heap at all. This means that there is much
less pressure on the GC, compared to something like the JVM. In particular, using dependent
types as shown above, one can make use of the compiler to track sizes when they are known
statically, and this can avoid the use of the heap altogether. I have used this technique
when working on a cryptography library. Since I did not make use of heap allocations, I
can compile it without including the GC at all, so that it is easier to use from other
languages, like a C library would.

Oh, and also the garbage collector of Nim [is written in Nim](https://github.com/nim-lang/Nim/blob/devel/lib/system/gc.nim),
which I think proves well the point that the language itself has complete control over
memory, regardless of being garbage collected.

At any rate, having garbage collection greatly simplifies the development of complex data
structures. This is why - like higher level languages - Nim has
[resizable sequences](http://nim-lang.org/docs/manual.html#types-array-and-sequence-types),
[hash tables](http://nim-lang.org/docs/tables.html), [sets](http://nim-lang.org/docs/sets.html)
and more.

Having a garbage collector also allows to have closures. While Nim is mostly imperative,
it is friendly to a functional programming style, thanks to higher-order functions,
[immutable variables](http://nim-lang.org/docs/manual.html#statements-and-expressions-let-statement)
and even an [effect system](http://nim-lang.org/docs/manual.html#effect-system) to track
exceptions, I/O and so on. A typical example would look like

~~~nimrod
import sequtils, future

type Person = object
  name: string
  friendliness: int
  # ...

let people = # ...

let friends = people.filter(p: Person => p.friendliness > 1).map(p: Person => p.name)
~~~

Benefits of Nim in scientific programming
-----------------------------------------

After this brief tour, I would like to explain why I think Nim may be a big deal
in scientific computing. The language is very high-level and readable, but still
most of what you write will run at C speed (in fact, it is usually straightforward
to figure out the resulting C).

At the same time, the sophisticated type system allows for very precise abstractions,
and together with macros it can form the basis of DSLs for various domains, such
as linear algebra, neural networks or symbolic programming. For instance, let me
take this example straight from the showcase for [Torch](http://torch.ch)

~~~lua
-- choose a dimension
N = 5

-- create a random NxN matrix
A = torch.rand(N, N)

-- make it symmetric positive
A = A*A:t()

-- make it definite
A:add(0.001, torch.eye(N))

-- add a linear term
b = torch.rand(N)

-- create the quadratic form
function J(x)
   return 0.5*x:dot(A*x)-b:dot(x)
end
~~~

In Nim it doesn't really look much more complex:

~~~nimrod
import linalg
# choose a dimension
const N = 5

# create a random NxN matrix
var A = randomMatrix(N, N)

# make it symmetric positive
A = A * A.t()

# make it definite
A += 0.001 * eye(N)

# add a linear term
let b = randomVector(N)

# create the quadratic form
proc J(x: Vector64[N]): Vector64[N] = 0.5 * x * A * x - b * x
~~~

Notice that the Nim code validates the dimension at compile
time and saves a few BLAS calls!

Of course, there is the problem of the lack of libraries, and it
will take time before Nim offers a mature environment for scientists.
Still, it is trivial to wrap existing C libraries, and there is even
a [tool](http://nim-lang.org/docs/c2nim.html) to automate the creation
of bindings, at least in part. This allows to rely on existing C libraries
and grow the ecosystem little by little.

What is still missing
---------------------

To be fair, there still are some downsides to be able to use Nim
productively. The most prominent issue is the lack of a working REPL.
There is the possibility of using [Nim for scripting](http://nim-lang.org/0.11.3/nims.html),
but the interpreter is not interactive, and there is no FFI to C,
so what one can do in this environment is pretty limited to the
standard library and a little I/O.

It should be possible to generate an interpreter having preloaded
a few more C modules, but the interpreter API is still being worked
out, and it is probably not trivial. But even then, the interpreter is
not interactive. There exists a hidden REPL, but it is not something
one can rely on, at least today.

Apart from the REPL, things are generally pretty smooth, but there are
occasional compiler bugs - certainly one can expect to meet a few of
them in the first few weeks working with Nim. Usually, this is not a big
deal for final users - most bugs are about something that should compile,
but doesn't, and it is enough to rewrite the thing in a slightly different
way - but it can be annoying to library authors, that may want to offer
a particular DSL to users, and so are less open to doing things differently
(as this would change the public API).

In the next few months I expect most bugs to iron out (bugs do not stay
open for long) and hopefully a REPL will be developed one day - certainly
after Nim 1.0 comes out.

I hope the above is enough to entice you and make you start contribute
to the Nim ecosystem: it is an exciting place to be, as lot of work needs
to be done, but it is very rewarding!
