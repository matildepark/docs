+++
title = "1.8 Doors"
weight = 16
template = "doc.html"
aliases = ["/docs/learn/hoon/hoon-tutorial/doors/"]
+++

It's useful to have [cores](/docs/glossary/core/) whose [arms](/docs/glossary/arm/) evaluate to make [gates](/docs/glossary/gate/).  The use of such cores is common in Hoon; that's how the functions of the Hoon standard library are stored in the subject.  Learning about such cores will also deepen the reader's understanding of Hoon semantics, and for that reason alone is worthwhile.

In this lesson you'll also learn about a new kind of core, called a 'door'.

## Two Kinds of Function Calls

There are two ways of making a function call in Hoon.  First, you can call a gate in the subject by name.  This is what you did with `inc` in the last lesson; you bound `inc` to a gate that adds `1` to an input:

```
> =inc |=(a=@ (add 1 a))

> (inc 10)
11

> =inc
```

The second way of making a function call involves an expression that _produces_ a gate:

```
> (|=(a=@ (add 1 a)) 123)
124

> (|=(a=@ (mul 2 a)) 123)
246
```

The difference is that `inc` is an already-created gate in the subject when we called it.  The latter calls involve producing a gate that doesn't exist anywhere in the subject, and then calling it.

Are calls to `add` and `mul` of the Hoon standard library of the first kind, or the second?

```
> (add 12 23)
35

> (mul 12 23)
276
```

They're of the second kind.  Neither `add` nor `mul` resolves to a gate directly; they're each arms that _produce_ gates.

Often the difference doesn't matter much.  Either way you can do a function call using the `(gate arg)` syntax.

It's important to learn the difference, however, because for certain use cases you'll want the extra flexibility that comes with having an already produced core in the subject.

## A Gate-Building Core

Let's make a core with arms that build gates of various kinds.  As we did in a previous lesson, we'll use the `|%` rune.  Feel free to cut and paste the following into the dojo:

```hoon
> =c |%
  ++  inc  |=(a=@ (add 1 a))
  ++  add-two  |=(a=@ (inc (inc a)))
  ++  double  |=(a=@ (mul 2 a))
  ++  triple  |=(a=@ (mul 3 a))
  --
```

Let's try out these arms, using them for function calls:

```
> (inc:c 10)
11

> (add-two:c 10)
12

> (double:c 10)
20

> (triple:c 10)
30
```

Notice that each arm in core `c` is able to call the other arms of `c` -- `add-two` uses the `inc` arm to increment a number twice.  As a reminder, each arm is evaluated with its parent core as the subject.  In the case of `add-two` the parent core is `c`, which has `inc` in it.

### Mutating a Gate

Let's say you want to modify the default sample of the gate for `double`.  We can infer the default sample by calling `double` with no argument:

```
> (double:c)
0
```

Given that `a x 2 = 0`, `a` must be `0`.  (Remember that `a` is the face for the `double` sample, as defined in the core we bound to `c` above.)

Let's say we want to mutate the `double` gate so that the default sample is `25`.  There is only one problem: `double` isn't a gate!

```
> double.c(a 25)
-tack.a
-find.a
```

It's an arm that produces a gate, and `a` cannot be found in `double` until the gate is created.  Furthermore, every time the gate is created, it has the default sample, `0`.  If you want to mutate the gate produced by `double`, you'll first have to put a copy of that gate into the subject:

```
> =double-copy double:c

> (double-copy 123)
246
```

Now let's mutate the sample to `25`, and check that it worked with `+6`:

```
> +6:double-copy(a 25)
a=25
```

Good.  Let's call it with no argument and see if it returns double the value of the modified sample.

```
> (double-copy(a 25))
50
```

It does indeed.  Unbind `c` and `double-copy`:

```
> =c

> =double-copy
```

Contrast this with the behavior of `add`.  We can look at the sample of the gate for `add` with `+6:add`:

```
> +6:add
[a=0 b=0]
```

If you try to mutate the default sample of `add`, it won't work:

```
> add(a 3)
-tack.a
-find.a
```

As before with `double`, Hoon can't find an `a` to modify in a gate that doesn't exist yet.

## Other Functions in the Hoon Standard Library

Let's look once more at the parent core of the `add` arm in the Hoon standard library:

```
> ..add
<74.dbd 1.qct $141>
```

The battery of this core contains 74 arms, each of which evaluates to a gate in the standard library.  This 'library' is nothing more than a core containing useful basic functions that Hoon often makes available as part of the subject.  You can see the Hoon code defining these arms near the beginning of [hoon.hoon](https://github.com/urbit/urbit/blob/master/pkg/arvo/sys/hoon.hoon), starting with [`++  add`](https://github.com/urbit/urbit/blob/master/pkg/arvo/sys/hoon.hoon#L21).  (Yes, the Hoon standard library is written in Hoon.)

Here are some of the other gates that can be generated from this core in the Hoon standard library.  It should be fairly obvious what they do:

```
> (dec 18)
17

> (dec 17)
16

> (gth 11 7)
%.y

> (gth 7 11)
%.n

> (gth 11 11)
%.n

> (lth 11 7)
%.n

> (lth 7 11)
%.y

> (lth 11 11)
%.n

> (max 12 14)
14

> (max 14 14)
14

> (max 14 432)
432

> (mod 11 7)
4

> (mod 22 7)
1

> (mod 33 7)
5

> (sub 234 123)
111
```

## Doors

A brief review: A core is a cell of battery and [payload](/docs/glossary/payload/): `[battery payload]`.  The battery is code and the payload is data.  The battery contains a series of arms, and the payload contains all the data necessary to run those arms correctly.

New material: A **door** is a core with a sample.  That is, a [door](/docs/glossary/door/) is a core whose payload is a cell of sample and context: `[sample context]`.

```
        Door
       /    \
Battery      .
            / \
      Sample   Context
```

It follows from this definition that a gate is a special case of a door.  A gate is a door with exactly one arm, named `$`.

Gates are useful for defining functions.  But there are many-armed doors as well.  How are they used?  Doors are quite useful data structures.  In Chapter 2 of the Hoon tutorial series you'll learn how to use doors to implement state machines, where the sample stores the relevant state data.  For now let's talk about how to use doors for simpler purposes.

### An Example Door {#example}

Let's write an example door in order to illustrate its features.  Each of the arms in the door will define a simple gate.  Let's bind the door to `c` as we did with the last core.  To make a door we use the `|_` rune:

```
> =c |_  b=@
  ++  plus  |=(a=@ (add a b))
  ++  times  |=(a=@ (mul a b))
  ++  greater  |=(a=@ (gth a b))
  --
```

If you type this into the dojo manually, make sure you attend carefully to the spacing.  Feel free to cut and paste the code, if desired.

Before getting into what these arms do, let's cover how the `|_` rune works in general.

#### The `|_` Rune

The `|_` rune for making a door works exactly like the `|%` rune for making a core, except it takes one additional subexpression.

The first subexpression after the `|_` rune defines the door's sample.  (This is the subexpression the `|%` rune lacks.)  Following that are a series of `++` runes, each of which defines an arm of the door.  Finally, the expression is terminated with a `--` rune.

We emphasize that a door really is, at bedrock level, the same thing as a core
with a sample. Let's ask dojo to pretty print a simple door.

```
> =a =>  ~  |_  b=@  ++  foo  b  --
> a
<1.zgd [b=@ %~]>
```

Dojo tells us that `a` is a core with one arm and a payload of `[b=@ %~]`. Since
a door's payload is `[sample context]`, this means that `b` is the sample and
the context is null. Setting the context is what the `=> ~` did. The reason we
did this was to avoid including the standard library that is included in the
context by default in dojo, which would have made the pretty printed core much
more verbose.

Now let's try to define a core with a sample:

```
> =a =>  ~  =|  b=@  |%  ++  foo  b  --
> a
<1.zgd [b=@ %~]>
```

Dojo gives us the same result! But is the pretty printer hiding something from
us? Recall that Hoon compiles down to Nock, Urbit's purely-functional
assembly-level language. The `!=` rune returns the Nock of any Hoon expression.
Let's try it on these expressions:

```
> !=  =>  ~  |_  b=@  ++  foo  b  --
[7 [1 0] 8 [1 0] [1 0 6] 0 1]
> !=  =>  ~  =|  b=@  |%  ++  foo  b  --
[7 [1 0] 8 [1 0] [1 0 6] 0 1]
```

You don't need to know any Nock to see that these two expressions are identical.
Thus we have witnessed that doors really are just cores with samples.

#### Back to the Example

For the door defined [above](#example), `c`, the sample is defined as an [atom](/docs/glossary/atom/), `@`, and given the face `b`.  The `plus` arm defines a gate that takes a single [atom](/docs/glossary/atom/) as its argument, `a`, and which returns the sum of `a` and `b`.  The `times` arm defines a gate that takes a single [atom](/docs/glossary/atom/), `a`, and returns `a` times `b`.  The `greater` arm defines a gate that takes a single [atom](/docs/glossary/atom/), `a`, and if `a` is greater than `b` returns `%.y`; otherwise `%.n`.

Let's try out the arms of `c` with ordinary function calls:

```
> (plus:c 10)
10

> (times:c 10)
0

> (greater:c 10)
%.y
```

This works, but the results are not exciting.  Passing `10` to the `plus` gate returns `10`, so it must be that the value of `b` is `0` (the bunt value of `@`).  The products of the other function calls reinforce that assessment.  Let's look directly at `+6` of `c`:

```
> +6:c
b=0
```

Having confirmed that `b` is `0`, let's mutate the `c` sample and then call its arms:

```
> (plus:c(b 7) 10)
17

> (times:c(b 7) 10)
70

> (greater:c(b 7) 10)
%.y

> (greater:c(b 17) 10)
%.n
```

Doing the same mutation repeatedly can be tedious, so let's bind `c` to the modified version of the door, where `b` is `7`:

```
> =c c(b 7)

> (plus:c 10)
17

> (times:c 10)
70

> (greater:c 10)
%.y
```

There's a more direct way of passing arguments for both the door sample and the gate sample simultaneously.  We may use the `~(arm door arg)` syntax.  This generates the `arm` product after modifying the `door`'s sample to be `arg`.

```
> (~(plus c 7) 10)
17

> (~(times c 7) 10)
70

> (~(greater c 7) 10)
%.y

> (~(greater c 17) 10)
%.n
```

Readers with some mathematical background may notice that `~( )` expressions allow us to [curry](https://en.wikipedia.org/wiki/Currying).  For each of the arms above, the `~( )` expression is used to create different versions of the same gate:

```
> ~(plus c 7)
< 1.gxk
  { a/@
    < 3.iba
      { b/@
        {our/@p now/@da eny/@uvJ}
        < 19.anu
          24.tmo
          6.ipz
          38.ard
          119.spd
          241.plj
          51.zox
          93.pqh
          74.dbd
          1.qct
          $141
        >
      }
    >
  }
>


> b:~(plus c 7)
7

> b:~(plus c 17)
17
```

Thus, you may think of the `c` door as a function for making functions.  Use the `~(arm c arg)` syntax -- `arm` defines which kind of gate is produced (i.e., which arm of the door is used to create the gate), and `arg` defines the value of `b` in that gate, which in turn affects the product value of the gate produced.

The standard library provides [currying
functionality](/docs/hoon/reference/stdlib/2n#curry) outside of the context of
doors - see `+curr` and `+cury`.

#### Creating Doors with a Modified Sample

In the above example we created a door `c` with sample `b=@` and found that the initial value of `b` was `0`, the bunt value of `@`. We then created new door from `c` by modifying the value of `b`. But what if we wish to define a door with a chosen sample value directly? We make use of the `$_` rune, whose irregular form is simply `_`. To create the door `c` with the sample `b=@` set to have the value `7` in the dojo, we would write
```
> =c |_  b=_7
  ++  plus  |=(a=@ (add a b))
  ++  times  |=(a=@ (mul a b))
  ++  greater  |=(a=@ (gth a b))
  --
```
Here the type of `b` is inferred to be `@` based on the example value `7`, similar to how we've seen casting done by example. You will learn more about how types are inferred in [Lesson 2.2](@docs/hoon/hoon-school/type-checking-and-type-inference.md).

### Doors in the Hoon Standard Library

Back in lesson 1.2 you were introduced to atom auras, which are metadata used by Hoon that defines how that atom is interpreted and pretty-printed.  Atoms are unsigned integers, but sometimes programmers want to work with fractions and decimal points.  Accordingly, there are auras for [floating point numbers](https://en.wikipedia.org/wiki/Floating-point_arithmetic).  Let's work with the aura for doing [single-precision](https://en.wikipedia.org/wiki/Single-precision_floating-point_format) floating point arithmetic: `@rs`.

The `@rs` has its own literal syntax.  These atoms are represented as a `.` followed by digits, and possibly another `.` (for the decimal point) and more digits.  For example, the float 3.14159 can be represented as a single-precision (32 bit) float with the literal expression `.3.14159`.

You can't use the ordinary `add` function to get the correct sum of two `@rs` atoms:

```
> (add .3.14159 .2.22222)
2.153.203.882
```

That's because the `add` gate is designed for use with raw atoms, not floating point values.  You can add two `@rs` atoms as follows:

```
> (add:rs .3.14159 .2.22222)
.5.36381
```

It turns out that the `rs` in `add:rs` is a Hoon standard library arm that produces a door.  Let's take a closer look:

```
> rs
<21|fan {r/?($n $u $d $z) <51.zox 93.pqh 74.dbd 1.qct $141>}>
```

The battery of this core, pretty-printed as `21|fan`, has 21 arms that define functions specifically for `@rs` atoms.  One of these arms is named `add`; it's a different `add` from the standard one we've been using for vanilla atoms.  So when you invoke `add:rs` instead of just `add` in a function call, (1) the `rs` door is produced, and then (2) the name search for `add` resolves to the special `add` arm in `rs`.  This produces the gate for adding `@rs` atoms:

```
> add:rs
< 1.hsu
  {{a/@rs b/@rs} <21.fan {r/?($n $u $d $z) <51.zox 93.pqh 74.dbd 1.qct $141>}>}
>
```

What about the sample of the `rs` door?  The pretty-printer shows `r/?($n $u $d $z)`.  What does this mean?  Without yet explaining this notation fully, we'll simply say that the `rs` sample can take one of four values: `%n`, `%u`, `%d`, and `%z`.  These argument values represent four options for how to round `@rs` numbers:

```
%n -- round to the nearest value
%u -- round up
%d -- round down
%z -- round to zero
```

The default value is `%z` -- round to zero.  When we invoke `add:rs` to call the addition function, there is no way to modify the `rs` door sample, so the default rounding option is used.  How do we change it?  We use the `~( )` notation: `~(arm door arg)`.

Let's evaluate the `add` arm of `rs`, also modifying the door sample to `%u` for 'round up':

```
> ~(add rs %u)
< 1.hsu
  {{a/@rs b/@rs} <21.fan {r/?($n $u $d $z) <51.zox 93.pqh 74.dbd 1.qct $141>}>}
>
```

This is the gate produced by `add`, and you can see that its sample is a pair of `@rs` atoms.  But if you look in the context you'll see the `rs` door.  Let's look in the sample of that core to make sure that it changed to `%u`.  We'll use the wing `+6.+7` to look at the sample of the gate's context:

```
> +6.+7:~(add rs %u)
r=%u
```

It did indeed change.  We also see that the door sample uses the face `r`, so let's use that instead of the unwieldy `+6.+7`:

```
> r:~(add rs %u)
%u
```

We can do the same thing for rounding down, `%d`:

```
> r:~(add rs %d)
%d
```

Let's see the rounding differences in action.  Because `~(add rs %u)` produces a gate, we can call it like we would any other gate:

```
> (~(add rs %u) .3.14159265 .1.11111111)
.4.252704

> (~(add rs %d) .3.14159265 .1.11111111)
.4.2527037
```

This difference between rounding up and rounding down might seem strange at first.  There is a difference of 0.0000003 between the two answers.  Why does this gap exist?  Single-precision floats are 32-bit and there's only so many distinctions that can be made in floats before you run out of bits.

Just as there is a door for `@rs` functions, there is a Hoon standard library door for `@rd` functions (double-precision 64 bit floats), another for `@rq` functions (quad-precision 128 bit floats), and more.

### Mutating the `rs` Door

Can we mutate the `rs` door so that its sample is `%u`?  Let's try it:

```
> rs(r %u)
-tack.r
-find.r
```

Oops!  Why didn't this work?  Remember, `rs` isn't itself a door; it's an arm that produces a door.  The `rs` in `rs(r %u)` resolves to the nameless parent core of `rs`, and the search for `r` commences there.  But that face can't be found in that parent core -- it's not where we want to look.

It's better simply to use the `~(arm rs arg)` syntax to replace the value of the `rs` door sample with `arg`.
