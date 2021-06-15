+++
title = "Limbs and wings"
weight = 30
sort_by = "weight"
template = "sections/docs/chapters.html"
aliases = ["docs/reference/hoon-expressions/limb/"]
+++
One feature Hoon lacks: a context that isn't a first-class value. Hoon has no
concept of scope, environment, etc.  An expression has exactly one data source,
the **subject**, a noun like any other.

## [Limbs](/docs/hoon/reference/limbs/limb)

A limb is used to resolve to a piece of data in the subject.  A wing is thus a resolution path into the subject.

## [Wings](/docs/hoon/reference/limbs/wing)

In Hoon we use 'wings' to extract information from the subject.  A wing is a list of limbs (including a trivial list of one limb).
