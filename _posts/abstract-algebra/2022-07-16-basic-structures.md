---
layout: post
title: "Conceptual Overview of Basic Algebraic Structures"
date: 2022-07-16 23:00:00 -5
categories: math algebra
examples: "/assets/abstract-algebra/basic-structures"
---

If you haven't seen the previous post in this series, and want
an introduction to abstract algebra before getting into specific structures,
you should check out [What is Abstract Algebra, Anyway?]({{ page.previous_in_category.url | relative }}).

As a reminder, we'll be starting to explore the various single-domain, single binary operator
algebraic structures in the following heirarchy.

<img src="/assets/abstract-algebra/heirarchy.svg">

We'll start at the bottom and work our way up, talking about each arrow individually.
Let's build some intuition!

* TOC
{: toc}

## Sets: Setting the Stage

Experienced mathematicians should feel comfortable skipping this section. I will introduce some
basic concepts and notation, then move on.

As mentioned in the last post, before we can have structure, we need to have things to put structure on.
This is the _domain_ of the structure, and it will almost always be a set of some kind.

Sets aren't themselves very interesting. They are collections of things. They can be any things.
Even other sets! We write sets as a list of things inside curly braces, for example $$\{1, 2, 3\}$$.
Even though we write them as a list, keep in mind that sets have no structure at all. $$\{3, 2, 1\}$$
is the same set, because the order doesn't matter (order would be structure). All that matters is that
the elements are the same.

However within a set, the elements do not have to look like each other. For example, we can easily
write $$\{1, cat, \text{:)}\}$$. I'm not sure how this set is useful, but it is a set.

There are several well-known sets with special names. For example, the natural numbers are written
$$\mathbb N$$. Here's a quick list of some common sets of numbers.

* $$\mathbb N$$ - the natural numbers
* $$\mathbb Z$$ - the integers
* $$\mathbb Q$$ - the rational numbers
* $$\mathbb R$$ - the real numbers
* $$\mathbb C$$ - the complex numbers

If we want to say that some thing $$x$$ belongs to a set $$S$$, we write $$x \in S$$. When we have
a set containing things that are alike, we can call the set a "type" and say that its elements
"are that type." For example, there is a type of natural numbers, and 0 is a natural number.
When viewing a set this way, we would instead write $$x : S$$.

There's also a set usually written **1**, which is the set containing a single element. It doesn't
really matter what that element is, because you can easily view any set with only one element
through the lens of any other set with only one element.

Wait. What does that mean?

In the last post we discussed how we can view natural numbers and stacks of pennies as being
"the same thing." We did that by describing how we could view both of them as modeling the same
set of properties. Here, what we're trying to say is that if you have a set with one element, say
$$\{1\}$$, then anything you say about that set will immediately apply to another set, say $$\{\text{dog}\}$$,
which also only has one element. We make it apply by replacing any reference to $$1$$ with a reference
to "dog."

What that describes is a way to _translate_ from $$\{1\}$$ to $$\{\text{dog}\}$$. This is a function!
If we call the function $$\tt{translate}$$, then we can define it as $$\tt{translate}(1) = \text{dog}$$.
We would say that the _type_ of this function is from $$\{1\}$$ to $$\{\text{dog}\}$$. The notation
for this is simply $$\tt{translate} : \{1\} \to \{\text{dog}\}$$.

We can also go the other way, using $$\tt{translate'} : \{\text{dog}\} \to \{1\}$$, in the obvious
way. Since we can go in both directions without losing or gaining any information, then conceptually,
both sets must contain exactly the same information to begin with!

This concept of "containing the same information" is at the core of what we can do with abstract
algebra. We can prove properties about some model by showing that it contains the same information
as some other model where those properties are easier to prove - and proving the properties on that
model instead.[^1]

That's a bit of an abstract concept, so let's work an example.

Let's suppose we have a computer program which reads a simple expression, like "1 + 1," as a string,
and evaluates it. We'll let the expressions contain a pair of natural numbers and either "+" or "*".

It's pretty hard to do any meaningful work on a string. Turning "1 + 1" into "2" is not so simple!

However most programming languages provide functions like `parseInt : String -> int` and
`toString : int -> String`. If we restrict ourselves to strings that actually contain integers,
then this pair of functions "witnesses" that those strings and the set of integers contain the
same information, just like `translate` and `translate'` _witnessed_ that $$\{1\}$$ and
$$\{\text{dog}\}$$ do.

So lets use these functions to translate our strings into a domain we can work on more easily:
```python
class Op(Enum):
    Add = 1
    Times = 2

def parseExp(input: str) -> Tuple[int, Op, int]:
    left, opstr, right = input.split()
    op = Add if opstr == "+" else Times # for simplicity
    return int(left), op, int(right)

def toString(result: Tuple[int, Op, int]) -> str:
    return [ str(result[0])
           , "+" if result[1] == Add else "*"
           , str(result[2])
           ].join(' ')
```

These two functions witness that (well-formed!) expressions contain the same information
whether we represent them as strings or as our little custom datatype.

These types of bidirectional transformations are _extremely_ common in programming. Hopefully
that motivates the power of abstract algebra to help us think about and improve our programs.

So let's finish this out. Now to show our property (that we can evaluate expressions-as-strings),
we have to show that our new custom datatype has the same property (that we can evaluate them).
Now it's easy;
```python
def eval(exp: Tuple[int, Op, int]) -> int:
    match exp[1]:
        case Add:
            return exp[0] + exp[2]
        case Times:
            return exp[0] * exp[2]
```

And of course we can recover the result as a string with `str`, if we want to.

This kind of bidirectional transformation is called a _bijection_. In order to know that we have
the same information both ways, we need to know that the transformations in each direction preserve
the structure we are working with. Bijections don't have to preserve structure, but sets don't have
any structure to preserve anyway. A bijection that _does_ preserve structure is called an
_isomorphism_, from the Greek _iso-_, meaning "same" and _morphous_ meaning "form."

For sets, all bijections are isomorphisms. In abstract algebra, we care much more about isomorphisms
than general bijections.

## Magmas: Both Lava and Trees

The section title is mostly a joke about the more common usage of the term "magma," don't try
and read too much into it.

There's only one arrow in the diagram to talk about here - we get magmas from sets.

In order to get a magma from a set, we need to add an operation that operates on two elements
of the set, and produces a third. This operation is typically written $$\cdot$$.

If our carrier set is $$S$$, we can write the type of $$\cdot$$ as $$\cdot : S \times S
\to S$$. For those unfamiliar, $$S \times S$$ means the type of two things in $$S$$, or a pair
of things in $$S$$. For example, $$(1, 2) \in \mathbb N \times \mathbb N$$. If $$x,y \in S$$, we
write $$\cdot(x, y)$$ or equivalently (and preferably) $$x \cdot y$$.

The only property that a magma imposes on the operation is its type. It must work for _any_ two
elements of $$S$$, and the result must also be an element of $$S$$. The operation does _not_ have
to be associative, commutative, or anything else.

Most of the structures we will look at later will subsume magmas, meaning that anything which models
them is _also_ a model of a magma. For the sake of example, though, let's consider something which
is a magma and _only_ a magma: binary trees.

A binary tree is a data structure where at any point, we either refer to two smaller binary trees,
or to nothing. Some examples of this magma:

<center><img src="{{ page.examples }}/magma-examples.svg"></center>

Get out a piece of paper and consider what $$A \cdot B$$ looks like. What about $$B \cdot A$$?

They aren't the same! That means that $$\cdot$$ does not _commute_.

What about $$(A \cdot B) \cdot C$$? Compare that to $$A \cdot (B \cdot C)$$. They also aren't the same!
This means that $$\cdot$$ does not _associate_.

The only property that we are guaranteed by a magma is _closure_: if $$a$$ and $$b$$ are in the magma,
then $$a \cdot b$$ exists and is also in the magma.

## Semigroupoids: Honestly, These Never Come Up

Despite the section header, these are worth talking about if only as a stepping stone.

Once again, we're going to imbue our carrier set $$S$$ with a binary operation $$\cdot : S \times S \to S$$.
However, this time, we're going to put a different requirement on it. Instead of requiring it to be _total_,
we require it to _associate_. What that means is that there might be some elements of $$S$$ for which $$\cdot$$
is undefined.

However, if $$a \cdot b$$ exists, and $$(a \cdot b) \cdot c$$ exists, then both $$b \cdot c$$ and
$$a \cdot (b \cdot c)$$ must _also_ exist. Furthermore, $$(a \cdot b) \cdot c = a \cdot (b \cdot c)$$.

We can see a semigroupoid as a directed graph by looking at the graph in a different way than we did for magma.

This time, consider the nodes of the graph. There are arrows between them; let the set of all those arrows
form our carrier set. Remember, sets can be of strange things! An arrow from node $$A$$ to node $$B$$ gives us
a "path" to get from $$A$$ to $$B$$. There are _a lot_ of notations used for this by different authors, and
honestly I find most of them obscure[^2]. So let's use a simple one: The arrow from $$A$$ to $$B$$ will
literally be written $$A \to B$$ :).

Now we can define $$x \cdot y$$ for two arrows $$x$$ and $$y$$. First of all, remember that $$\cdot$$ does
not have to be defined for every pair of arrows. So since arrows describe paths between nodes, let's let
$$\cdot$$ be the operator which _combines_ paths. That seems pretty natural, right?

We define $$(A \to B) \cdot (B \to C) = A \to C$$. In order for a particular graph to to model a
semigroupoid in this way, the arrow $$A \to C$$ _must_ exist. That's not a property of semigroupoids though.
It's just a property of the way we have defined this model. Here's a pair of examples of such semigroupoids.

<center>
  <img src="{{ page.examples }}/semigroupoids.svg">
</center>

A very important facet of abstract algebra is apparent here: there are _many_ things that fit the mold of
"semigroupoids," and two of them are shown here, but these two semigroupoids are _not_ the same. Proving
something about the first would not necessarily prove the same thing about the second, because they don't
contain the same information! In math-speak, these semigroupoids are not _isomorphic_.

We could easily take some arbitrary set $$\{$$ &#128578;, &#128577;, &#128515; $$\}$$ and define, again arbitrarily,
that &#128515; $$\cdot$$ &#128577; = &#128578;. The result is certainly a semigroupoid (there aren't enough
equations to even trigger the associativity requirement). But this semigroupoid is _isomorphic_ to the first
one above (why?). Since they carry the same information, we only have to talk about one of them. Graphs
are easier to visualize, so we're going to use them!

In the second semigroupoid above, notice that $$2 \to 2$$ and $$4 \to 4$$ exist, because of the pair
of arrows $$2 \to 4$$ and $$4 \to 2$$. We can take $$(2 \to 4) \cdot (4 \to 2)$$ to "round-trip" back to
$$2$$, and according to our definition, that means $$2 \to 2$$ has to exist.

We can also use the second semigroupoid to check that the way we defined the operation is actually associative. You can check that

$$\begin{aligned} ((1 \to 2) \cdot (2 \to 4)) \cdot (4 \to 3) &= (1 \to 4) \cdot (4 \to 3)\\
&= 1 \to 3\\
\\
(1 \to 2) \cdot ((2 \to 4) \cdot (4 \to 3)) &= (1 \to 2) \cdot (2 \to 3)\\
&= 1 \to 3\\
\end{aligned}$$

Indeed, those are the same.

However, it's also quite apparent that $$\cdot$$ does not commute. In fact, it fails to commute in the most
spectacular fashion imaginable: not only does $$(B \to C) \cdot (A \to B)$$ fail to equal $$(A \to B) \cdot (B \to C)$$,
it doesn't even exist! We didn't define the operation for arrows that don't share an intermediate node, and the
semigroupoid laws don't say we have to.

Since arrows describe paths between nodes, it's pretty common to only draw the minimal number of arrows and leave
the combined paths as implied. In the complex graph, we could omit arrows $$2 \to 2, 4 \to 4, 1 \to 3, 2 \to 3,$$ and $$1 \to 4$$.

## Small Categories: It Begins

Small categories are where things really start getting interesting. For simplicity, I'm just going to call these
categories. This is conceptually fine, but you can see the footnote to see what "small" means here if you want.[^3]

With semigroupoids, we saw that it was ocassionally necessary to have an arrow $$a \to a$$, in order to make
sure that the product of two arrows exists. When we have these arrows, we get the nice pair of equations

$$\begin{aligned}
  (a \to a) \cdot (a \to b) &= a \to b \\
  (a \to b) \cdot (b \to b) &= a \to b \\
\end{aligned}$$

These are respectively called the left identity equation and the right identity equation. For the arrow
$$a \to b$$, the arrow $$a \to a$$ is called a left identity and the arrow $$b \to b$$ is called a right
identity.

A _category_ is what we get if we _require_ that these identity arrows exist. We add a pair of requirements
to our operation:

1. For any element $$x$$, there exists an element $$e_l$$ such that $$e_l \cdot x = x$$
2. For any element $$x$$, there exists an element $$e_r$$ such that $$x \cdot e_r = x$$

In our graph model[^4], recall that we only required arrows to exist if they were necessary to "combine
paths." These two new requirements additionally require arrows to exist from every node to itself.
Collectively, these requirements are called the "left and right identity laws." We're never going to want
one without the other, so we'll shorten that to just "the identity law," and use that phrase to mean the
pair of them together.

Categories have the nice property that for any node in the graph, there is _definitely_ an arrow pointing
away from it, and an arrow pointing towards it. That means that when proving things about categories, we
can freely say "pick an arrow pointing to $$b$$" or "pick an arrow from $$a$$," without running into the
problems that this could cause with a semigroupoid.

Other than our graph model, a pretty typical example of a category is that of functions from some type
to itself. If we consider the collection of all functions $$f : S \to S$$, we get a category. The operation
is function composition. The (singular) identity is simply $$f(x) = x$$, which is both a left and right
identity to every other function in the category. Such functions are often referred to as _endomorphisms_,
from the Greek root _endo-_ meaning "within." This structure is incidentally more than just a category.

## Semigroups (from Semigroupoids): The Graph Collapses

Every arrow in our original heirarchy[^5] represents adding a single new restriction to the operation $$\cdot$$.
This time, we're taking a semigroupoid (which requires that the operation _associate_) and additionally require
that the operation be _total_.

In our graph view, this means we have to somehow define $$(a \to b) \cdot (c \to d)$$, and it's not obvious how
to do that. In fact, it seems there isn't any reasonable way to do it at all!

There's one exception. If we allow our graph to have multiple arrows between the same nodes, then a graph
with only one node and many arrows from that node to itself forms a semigroup. Since every arrow looks like
$$(a \to a)$$, we can always define $$\cdot$$. But now it's not so obvious how this is useful.

We could imagine that the singular node represents some set, and that each arrow represents
some arbitrary function from that set to itself. This is _extremely_ general, but it is a semigroup.
Really, it's a whole family of semigroups, because it will form a semigroup no matter which functions
we choose from the set to itself. Notably, unlike the similar category example, we _don't_ have to have
the identity function in our collection!

Since semigroups are semigroupoids (and magmas), anything that we can prove about semigroupoids (or magmas)
will also apply to semigroups, and therefore applies to this amazingly general structure of "composing
functions from a set to itself."

## Semigroups (from Magmas): Just Change Equality

We can get an alternate view of semigroups by coming from Magmas. Magmas required the operation to
be total. Semigroups additionally require the operation to associate.

Looking back at [the section on magmas]({{ page.url | relative }}#magmas-both-lava-and-trees), we
called two trees equivalent if they looked the same. This was the reason the operation didn't associate.
But what if we redefine equality, specifically so that $$a \cdot (b \cdot c) = (a \cdot b) \cdot c$$?
No one says we _can't_ do this. What structure do we end up with?

Since we can now shift the parentheses around arbitrarily, we can always move them as far to the left
as we want. The result will be binary trees that look like this:

<center>
  <img src="{{ page.examples }}/semigroup-chain.svg">
</center>

So we now say two trees are equivalent if we can rearrange their nodes (but without swapping the _order_ -
semigroups still aren't commutative!) to move all the "interesting" nodes to the left, and they look the
same after we're done. If we wanted to worry about order, we could label the nodes at the bottom with numbers,
but let's keep it simple here and leave them as dots.

We can see an easier criteria for equality based on that. Two of these semigroups are "the same" if they have
the same number of nodes. That's it!

And since we can always adjust such a tree to whatever (tree) structure we like, the "structure" that the
semigroup knows is just the order of the leaves at the bottom. That carries exactly the same information
as an ordered list of the leaves![^6] Notice though, that this model doesn't support empty lists (yet).

So by coming at semigroups from different angles, we are able to see two _completely_ different views of
the object. And yet amazingly, both views describe exactly the same thing. This is the power of abstract
algebra at work. We can prove things about semigroups, mentally reasoning about them as nonempty lists, and
as long as in practice we only use the semigroup laws, we will get the same results about semigroups as
graphs for free! _Including_ the semigroup of function compositions that we saw in the last section,
which is seemingly entirely unrelated to the semigroups we are looking at now. In fact, they are one
and the same!

This relationship can be daunting. How can we conceptualize that they have the same structure?

I think it is best to simply not try and find deeper meaning here. We've seen that what matters is
the _properties_ that they have in common. In both cases, we have a set, and a binary operation which is total and
associative. This is all the relationship that they need.

Rather than be confused how they can be the same, I think it suffices to see this core property that they
share and simply sigh in awe.

## More To Come

I'm going to leave this post off here to avoid it getting _too_ long. In the next post we will take a brief
look at unital magma and quasigroups. Then we'll get to the interesting stuff that comes up in programming
and in life all the time - monoids and groups.

--------

[^1]: It is really important that we show the models contain the same information. We could have
      two different models of a structure which _do not_ contain the same information, and that difference
      in information might make the properties true in one model but false in the other!

[^2]: The worst offender for newcomers is $$Hom(A, B)$$ for $$A \to B$$. This comes from category theory
      and I won't be using it here. But if you do see it in the wild, mentally rewrite it to $$A \to B$$.

[^3]: Essentially, there's a problem with the way we hand-wavily defined a set as a collection of anything.
      We could define a set as "the collection of all objects with some property," for example all numbers
      that are natural, or all integers divisible by 3. But even worse, we can define sets of sets in some
      ways that are straight-up impossible. The quintessential example is called Russel's Paradox. The set
      of "all sets that do not contain themselves" cannot exist. If it doesn't contain itself, it should.
      But if it contains itself, then it shouldn't! This was a major problem in the formalization of math,
      and modern set theory systems define sets in a way that prevents this. But category theorsists _really_
      want to be able to talk about categories of categories in the most generic way possible. To solve this
      problem, we distinguish between "small" categories, whose arrows (and objects) form sets in the modern
      set-theory way, and "large" categories, whose arrows (and objects) are just defined by some property.
      The latter collections are called "proper classes," and generally a "class" is a collection of things
      which _are not_ classes. All this to say, the word "small" here solves a problem that we didn't even
      have to think about, so I'm just going to drop it in this post.

[^4]: The graph model is actually the standard model of categories. Categories are usually defined as having
      _two_ carrier sets, the graph nodes and the graph arrows, considered separately. But for our purposes
      it's really enough to consider just the arrows and assume that they point between distinguishable things.
      Either way, if you had a different model of a category, it would be possible to transform it to the
      graph model and only _lose_ information, not gain anything. Category theorists say the graph model is
      "universal," but in our language, we'll say graphs are "the free categories on graph nodes." It
      will be a while before we get into free structures, but it is one of my eventual goals in this post
      category.

[^5]: Does that sound suspiciously like a category? Because it is!

[^6]: From a programming perspective, this relationship is what lets us turn programs (strings of words)
      into richer _parse trees_. The programming language dictates what additional non-associative tree
      structure to imbue the text with, but the original program needn't care. It's just a list!