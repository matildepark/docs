+++
title = "Nock"
weight = 30
template = "doc.html"
+++


Nock is a low-level [homoiconic](https://en.wikipedia.org/wiki/Homoiconicity)
combinator language.  It's so simple that its [specification](/docs/nock/definition)
fits on a t-shirt.  In some ways Nock resembles a nano-Lisp but its ambitions
are more narrow.  Most Lisps are one-layer: a practical language is to be
created by extending a theoretically simple interpreter.  The abstraction is
simple and the implementation is practical.  Unfortunately it's far more difficult
to enforce both simplicity and practicality in an actual Lisp codebase.  Hoon
and Nock are two layers: Hoon, the practical layer, compiles itself to Nock, the
simple layer.  Your urbit runs in Vere, which includes a Nock interpreter, so it
can upgrade Hoon over the network without downtime.

The Nock data model is quite simple.  Every piece of data is a 'noun'.  A [noun](/docs/glossary/noun/)
is an [atom](/docs/glossary/atom/) or a cell.  An atom is any unsigned integer.  A cell is an ordered
pair of nouns.  Nouns are acyclic and expose no pointer equality test.

## Further Reading

* [Nock Definition](/docs/nock/definition): The Nock specification.
* [An explanation of Nock](/docs/nock/explanation): A comprehensive
  walkthrough of the Nock spec.
* [Nock by hand](/docs/nock/example): Learn Nock by example.
* [Nock Implementations](/docs/nock/implementations): The many ways that
  Nock has been implemented.
* [Nock for Everyday Coders](https://blog.timlucmiptev.space/part1.html): A
  comprehensive guide to Nock by community member `~timluc-miptev`.


