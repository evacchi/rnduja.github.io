---
layout:     post
title:      "Dependent types in Scala"
subtitle:   "Hacking on the type level"
date:       2015-10-07 12:00:00
author:     "Andrea Ferretti"
header-img: "img/2015-10-07-scala-dependent-types/cover1.jpg"
---

This is the iScala notebook for the talk I presented at the [Scala Italy](http://www.scala-italy.it/) conference.

**Abstract.** An exploration of type level computations. First, I would introduce a few standard techniques to summon implicit instances for a given type, or detect type equality. Then, I would show how to use the type system to encode values and computations, with the examples of booleans and Peano naturals. Finally, the main goal of the talk would be to show how these type level encodings allow Scala to partially simulate some typical constructions of dependently-typed languages, such as fixed-size sequences, modular arithmetic and balanced trees. Many of these constructions have been explored before, and good references are the blog post series by Mark Harrah on type-level programming and the library Shapeless by Miles Sabin. The whole presentation is meant to be done inside an interactive iScala notebook, which can then be distributed to participants.

<iframe src="https://player.vimeo.com/video/131975223" width="500" height="268" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe> <p><a href="https://vimeo.com/131975223">Scala Italy 2015: Andrea Ferretti - Dependent types in Scala</a> from <a href="https://vimeo.com/user41133250">Scala Italy</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

<p>
  <a href="/files/type-level-hackery.ipynb"> download notebook</a>
</p>

<iframe width="100%" height="500px" frameborder="0" src="http://nbviewer.ipython.org/github/scala-italy/2015/blob/master/05a-andrea_ferretti-type-level-hackery/05a-andrea_ferretti-type-level-hackery.ipynb"></iframe>
