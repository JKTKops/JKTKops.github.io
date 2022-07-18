---
layout: post
title: "More Basic Algebraic Structures"
date: 2022-07-18 00:30:00 -5
categories: math algebra
examples: "/assets/abstract-algebra/basic-structures"
---

In [the last post]({{ page.previous_in_category.url | relative }}), we saw how we
can view the heirarchy of algebraic structures as a system of adding requirements
to the binary operator of each structure.

<img src="/assets/abstract-algebra/heirarchy.svg">

In this post we'll wrap up our exploration of the bottom half with unital magma
and quasigroups. Then we'll start talking about the big guns: monoids.

* TOC
{: toc}

# Unital Magma: One Single Lava, Please

The possible properties we've put on the operator so far include totality,
association, and identities[^1]. We got semigroups by starting from magma and
requiring the operation to be associative. What if instead, we require the
operation to have identities?

Specifically, the property we're going to add is slightly different than what we
had before. Before we assumed that there was an identity for _each_ arrow in
a category, and that the left and right identities could be different. However,
this was only really necessary because categories did not require the operation to
be total.

The reason this is undesirable is because that if I give you an element of the set
$$S$$ and ask you for the (left or right) identity element, you can't easily give
it to me. You'd have to find the correct identity _for that element_, and this could
be quite difficult. So we're going to add a stronger requirement instead, which
makes it easy to find the correct identity.

$$\text{The Identity Axiom} \\ \text{There is an } e \text{ such that for all } x, e \cdot x = x \cdot e = x$$

Now it's easy to find the correct identity, because it's always the same.

We do have to be careful though. The axiom doesn't say that there is _only one_ $$e \in S$$
with this property, just that there is _at least_ one. We cannot (naively) assume
that the identity is unique - we'll come back to this in a moment.

Recall that we defined $$\cdot(A, B)$$ for our model of magma to be the operation
that makes a new tree node whose children are $$A$$ and $$B$$. The carrier set
for this model is the set of (rooted) binary trees. But I implicitly disregarded
the existence of _empty_ trees - we assumed that every tree in the set has at
least one node. This was effectively required because every node needs to have either
zero or two children; $$\emptyset \cdot A$$ would only have one child which is not
allowed.

But we can still give a definition for $$\emptyset \cdot A$$ - it can just be $$A$$
again! By definition, $$\emptyset$$ (the empty tree) would be our left identity.

We can similarly define $$A \cdot \emptyset = A$$, and now it's also the right identity[^2].

This lets $$\emptyset$$ satisfy our definition of the identity element.

Now recall that we got here by adding $$\emptyset$$ to our carrier set. It wasn't there
before, but we've added it in and defined $$\cdot$$ for it. What's stopping us from
also adding a new element, &#128578;, and defining it to _also_ be an identity?

Seemingly, the answer is nothing. And this takes us to our first highly-general application
of the ideas of abstract algebra.

### Identity Elements Are Unique!

Let's suppose that we have a unital magma with two _distinct_ elements $$e_1, e_2$$ which
both satisfy the identity law. What happens if we try and compute $$e_1 \cdot e_2$$?

$$\begin{aligned}
e_1 &= e_1 \cdot e_2 \\
&= e_2
\end{aligned}$$

We can prove that $$e_1 = e_2$$! This happens because both identities are _both_ left and
right identities.

The amazing thing is that this proof only uses the identity law. Whenever we have a
structure with this identity law, we get _for free_ that the identities are unique!

This applies to anything that is a unital magma - unital magmas, but also monoids, loops,
groups, and more.

Therefore, from now on, we will say _the_ identity instead of _an_ identity.

# Quasigroups: Latin Squares

Latin Squares are a generalization of Sudoku puzzles. We have an $$n \times n$$ grid,
and $$n$$ distinct symbols. We place each element in the grid such that each one appears
exactly once in every row an column. Here's an example one with 4 elements - we say it
has _order_ 4.

$$\begin{array}{|c|c|c|c|}
\hline
b & d & a & b \\
\hline
a & c & b & d \\
\hline
c & b & d & a \\
\hline
d & a & c & b \\
\hline
\end{array}$$

It's not very hard to check that this is indeed a latin square. Take a few moments
and check!

What's interesting, though, is that this looks suspiciously like a multiplication
table. All we have to do is label the rows and columns with the operands and then
declare that $$x \cdot y$$ is the value in the row labeled $$x$$ and column labeled $$y$$.

$$\begin{array}{c||c|c|c|c|}
  & a & b & c & d \\
\hline\hline
a & b & d & a & b \\
\hline
b & a & c & b & d \\
\hline
c & c & b & d & a \\
\hline
d & d & a & c & b \\
\hline
\end{array}$$

We could describe any magma in this form, but our running example so far has used the domain
of binary trees, which is an infinite domain. Putting that in table form would be pretty tricky!
However the magma described by this table is finite. The carrier set is just $$D = \{1,2,3,4\}$$.

$$D$$, along with the multiplication table to define $$\cdot(-,-)$$[^3]. Since the operation is
defined by a multiplication table, we could instead call the operation multiplication. Let's do
that. While we're at it, we might as well use the multiplication symbol $$*$$.

For example, we have $$a * b = d$$. This pair of $$(D, *)$$ gives us a "finite magma."

Take a moment and check: is this magma unital? Answer[^4]

We should also check if this magma is _associative_, which would make it a semigroup. Randomly
picking elements $$b, a, d$$, we can check:

$$\begin{aligned}
(b * a) * d &= a * d \\
            &= b \\
\\
b * (a * d) &= b * b \\
            &= c \\
\end{aligned}$$

It's not associative!

However, the fact that the multiplication table is a latin square gives us an interesting
property. The multiplication is _invertible_. That is, we have the following property.

$$\text{The Divisibility Axiom} \\ 
\text{For any } a,b \in D, \text{ there are unique } x,y \in D \text{ such that} \\
a * x = b \\
y * a = b$$

We write that $$x = a \backslash b$$ and $$y = b / a$$. I read these as "x is a under b" and
"y is b over a." Respectively, we call these operations "left division" and "right division."

If a magma has the divisibility property, we could call the magma _divisible_. But more commonly,
we call these structures _quasigroups_.

We can check, from the definitions, that all of the following properties hold:

$$\begin{aligned}
y &= x * (x \backslash y) \\
y &= x \backslash (x * y) \\
y &= (y / x) * x          \\
y &= (y * x) / x          \\
\end{aligned}$$

These identities say that multiplication and division on the same side, by the same element, in
either order, have no effect. We'd expect that of operations called "multiplication" and
"division," and we get these identities _for free_ from the definitions even though we didn't
require them explicitly!

These properties will hold in any quasigroup. Checking the diagram, groups are (unsurprisingly)
quasigroups, and groups are very prevalent. These properties will also hold in groups.

The most common quasigroups are numbers with subtraction, for example $$(\mathbb Z, -)$$ or
$$(\mathbb R, -)$$. Subtraction is total, and invertible on either side. We don't typically
think of subtraction as "multiplication," but it fits the definition. And indeed, the above
properties hold in these quasigroups.

# Loops: A Quick Detour

If we add the identity axiom to our requirements for a quasigroup, we get a structure called
a _loop_. One example of a loop (which is not also associative) is described by the following table.

$$\begin{array}{c|c|c|c|c}
1 & 2 & 3 & 4 & 5 \\
\hline
2 & 4 & 1 & 5 & 3 \\
\hline
3 & 5 & 4 & 2 & 1 \\
\hline
4 & 1 & 5 & 3 & 2 \\
\hline
5 & 3 & 2 & 1 & 4 \\
\end{array}$$

There are some more subclassifications of loops, but I'm not going to get into them here. I just
wanted to mention that this structure has a name, and it's going to be one of the possible
stepping stones to groups.

# Groupoids: Invertible Categories

Continuing our exploration of inverses, what happens if we add inverses to a category? That is,
for every arrow $$A \to B$$ in a category, we'll add the arrow $$B \to A$$ if it doesn't already
exist.

The exact properties we'll add (which agree with our notion of division from quasigroups) are

1. For every element $$a \in D$$, there exists an element $$a^{-1} \in D$$.
2. $$a^{-1} \cdot a$$ and $$a \cdot a^{-1}$$ are defined (but not necessarily equal).
3. If $$a * b$$ is defined, then $$a * b * b^{-1} = a$$ and $$a^{-1} * a * b = b$$.

The third property is a version of the identity property that works despite the fact that
groupoids don't actually have to have identities. It says that whatever the result of
$$a * a^{-1}$$ is, it is an identity for any values with which multiplication _is_ defined.
However, not every element necessarily has multiplication defined with an identity!

In every groupoid, we can prove the following useful theorems:

1. For any $$a$$, $$(a^{-1})^{-1} = a$$
2. If $$a * b$$ exists, then $$(a * b)^{-1} = b^{-1} * a^{-1}$$.

The proofs require some strong symbolic manipulation. This is pretty common in abstract
algebra (hence the "abstract") but the exchange is that the results are very general and
powerful. Let's get in the practice.

### Proof Of Inverse-Inverses

$$\begin{aligned}
a * a^{-1} &= a * a^{-1} & \text{Equality is reflexive} \\ 
(a * a^{-1}) * (a^{-1})^{-1} &= (a * a^{-1}) * (a^{-1})^{-1} & \text{Right-multiply by } (a^{-1})^{-1} \\
(a^{-1})^{-1} &= (a * a^{-1}) * (a^{-1})^{-1} & \text{New Axiom 3} \\
(a^{-1})^{-1} &= a * (a^{-1} * (a^{-1})^{-1}) & \text{* is associative} \\
(a^{-1})^{-1} &= a & \text{New Axiom 3} \\
\end{aligned}$$

The right multiplication is justified because associativity and axiom 2 tell us that the
relevant product is defined.

### Proof of Inverse-of-Product

This proof looks very similar to the last one. We start with the only relevant equality that
we know, and then we rearrange things in the only two ways possible and the equality we want
just falls out.

Given that $$a * b$$ exists, we know that $$(a * b)^{-1}$$ exists. Let's call it $$e$$.
Similarly to the last proof, we can justify that $$a * b * b^{-1} * a^{-1}$$ exists.

$$\begin{aligned}
e &= (a * b)^{-1} & \text{Definition of } e \\
e * (a * b) * (b^{-1} * a^{-1}) &= (a * b)^{-1} * (a * b) * (b^{-1} * a^{-1}) & \\
e * (a * b) * (b^{-1} * a^{-1}) &= (a * b)^{-1} * (a * (b * b^{-1})) * a^{-1} & \text{Associativity} \\
e * (a * b) * (b^{-1} * a^{-1}) &= (a * b)^{-1} * a * a^{-1} & \text{Axiom 3} \\
e * (a * b) * (b^{-1} * a^{-1}) &= (a * b)^{-1} & \text{Axiom 3} \\
(a * b)^{-1} * (a * b) * (b^{-1} * a^{-1}) &= (a * b)^{-1} & \text{Definition of } e \\
(b^{-1} * a^{-1}) &= (a * b)^{-1} & \text{Axiom 3} \\
\end{aligned}$$

Both of these properties are easier to prove on proper groups, but it's interesting that we
don't actually need the existence of an identity to prove them.

### The Category Model

Returning to our category model, a groupoid is a category with inverses. Using our function
example, each node of the graph is a set. Each arrow $$A \to B$$ is a function from $$A$$ to $$B$$.
For simplicity, we're only going to pick _one_ function for each arrow.

If we have an arrow $$f : A \to B$$, we also have an arrow $$g : B \to A$$. For each pair, we
know that both products $$f * g$$ and $$g * f$$ are defined. This is our Axiom 2 of inverses
above, and we can easily check that the model meets it: $$(A \to B) * (B \to A) = (A \to A)$$
and vice versa.

We can also check the properties that we proved in the last couple sections, but I'll leave
that as an exercise.

### A More Interesting Example

You may have heard people say that group theory can be applied to study Rubik's Cubes. This is
true, and part of the reason is that any two moves on a Rubik's Cube can be composed. Each move
is an element of the "Rubik's Cube Group," and products in a group always exist. Of course,
this group is also a groupoid.

But we could find a different puzzle where we can't always compose any moves, for example,
[fifteen puzzles](https://en.wikipedia.org/wiki/15_puzzle). Fifteen puzzles are commonly
studied as groups, but they can be more naturally represented by the groupoid of sequences of
moves. The product of two sequences means doing one and then the other (and since this is only
possible if the hole is in the correct place in the intermediate configuration, the product
does not always exist).

We'll still have all of the properties that we've shown hold for general groupoids. We have
partial identities (do nothing, with the hole in a particular place), inverses,
and composition associates. We could then apply
_groupoid_ theory to the fifteen puzzle to discover various properties of the
"15 puzzle groupoid" and learn things about the nature of the puzzle.

One common fact about 15 puzzles is that exactly half of the possible configurations are
solvable. Any solvable configuration can be transformed to the solved configuration by
applying some element of the 15-puzzle groupoid[^5] (a sequence of moves). It's easy to
prove, using the groupoid axioms, that any two solvable configurations can be transformed
into each other (exercise). If we went and developed some groupoid theory, we could show
that these facts _imply_ that any two _unsolvable_ configurations can also be transformed
into each other, which is much less obvious.[^6]

The development of such a theory may be the topic of future posts, but
this hopefully teases some of the power of abstract algebra. We get general results
which we can then apply to learn specific things about specific models.

# Groupoid Tangent: Torsors

An extremely common structure in the real world is a _torsor_. Torsors are sets,
equiped with an invertible binary operation (often written $$+$$), but without
a notion of "zero." This corresponds to the notion of a groupoid, although if
we make it more precise we'll see that I'm waving my hands fairly vigorously
at the moment.[^7]

But these posts are about conceptually understanding structures, so, I'll keep
waving my hands for the moment.

Consider musical notes. We have notes like A, A#, D, Fb, etc. There's a notion
of a "next note," and of a "previous note," `next(A) = A#`, `prev(A) = Ab`. These
operations are inverses. If we see the notes as the nodes of a graph, then the
`next` and `prev` functions are arrows between the nodes, and we have a groupoid.

Using these functions, we can measure _distances_ between notes. It takes 4 steps
of the `next` function to get from A to C#. We can say that the distance between
these notes is 4. By measuring distances, we can recover something that looks
a lot like addition. However, we don't have values that we can assign to notes themselves,
other than arbitrary names, and we don't have a notion of a "zero" note. So we
can add a note to a _distance_, for example `B + (C# - A) = D#` (why?), but we can't
add notes to other notes.

Torsors are pretty common in the real world. In physics, energy is a torsor. There's
no notion of "zero" energy. If we examine the same scenario in two different reference
frames, we will almost certainly measure different amounts of energy for every object
involved. Which measurement of "zero" is the right one? Neither! They are both
equally meaningless. Yet both analyses will come to exactly the same conclusions, because
they will measure the same _differences_ in energy, and this is what matters.

Sometimes we think something is a torsor and later find out that there is a true zero.
Temperatures are a good example. Temperatures were thought to be a torsor until
absolute zero was discovered. Absolute zero is the true zero of temperatures. No matter
what scale we use to measure temperatures (analogous to a reference frame), we will
always agree on the meaning of absolute zero.

# Next Time: Monoids

This concludes the introduction to the basic algebraic structures, and motivating some
of the things that they gives us the language to talk about. In the next post, we will
talk about _monoids_, which are by far the most prevalent mathematical structure in programs
that you've never heard of[^8]. We'll also talk about groups, which are a prevalent
_mathematical_ structure. 

------

[^1]: Recall from the previous post's section on categories, we say "the identity
      law" to mean the existence of _both_ left and right identities.

[^2]: The identity laws for a small category require that an identity exists _for each_
      element of the carrier set. They _do not_ require that the identities be unique
      or the same for every element. 

[^3]: This notation means "the operation $$\cdot$$ which takes two unspecified arguments."
      I noticed that plain $$\cdot$$ is a bit awkward to read on the rendered page, so
      I'll use this from now on when referring to an operation that we don't have a better
      name for.

[^4]: No, it is not. There's no left _or_ right identity, let alone something which is both.

[^5]: I didn't define what it means to apply a sequence of moves to the puzzle, but
      intuitively we know what it means. However it's worth noticing that this is
      itself a binary operation. Rather than being from $$D \times D \to D$$, though,
      it is from $$F \times D \to F$$, where $$F$$ is the set of 15-puzzle configurations.
      Such operations are called _groupoid actions_. These are closely related to
      _group actions_, and many of the same results apply to both.

[^6]: Group theorists will recognize this as the statement that the groupoid action which
      applies a sequence of moves to a configuration has two orbits, with equal cardinality.

[^7]: Torsors are usually analyzed as a _group_ acting on a different set $$X$$. Using the
      group action and the group multiplication (which _does_ have an identity), we can
      obtain a subtraction operation on $$X$$ which measures distances as elements of the
      group. The 0 distance corresponds to the identity of the group.
      Yet the set $$X$$ does not have a designated zero. If we pick an arbitrary element of
      $$X$$ and declare it to be zero, we recover the group itself, and this is called
      "trivializing" the torsor. But seeing things as a torsor is useful when we want to
      work directly and only on the distances between things, and I find this conceptually
      close to groupoids. Your mileage may vary with this one.

[^8]: Unless you enjoy programming in Haskell, in which case you probably recognize that
      just about everything in programs is a monoid &#128578;
