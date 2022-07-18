---
layout: post
title: "What are Monoids and Groups?"
date: 2022-07-18 17:00:00 -5
categories: math algebra
---

In [the last post]({{ page.previous_in_category.url | relative }}), we explored
unital magma, quasigroups, loops, and groupoids. Each structure arose by adding
more restrictions to the binary operation of the structure.

<img src="/assets/abstract-algebra/hierarchy.svg">

When we looked at groupoids, we saw how the combination of restrictions can lead
to generalized results about _all_ groupoids, like the inverse of the inverse
theorem, and the inverse of product theorem.

In this post, we're going to start looking at monoids, which have a set of properties
satisfied by, well, nearly everything. In particular, monoids appear _everywhere_
in programs.

* TOC
{: toc}

# Monoids (From Categories): Totality

Categories didn't have a requirement that multiplication be total. For many pairs
of arrows, it doesn't make sense to chain them together. It only works if they have
the same intermediate node in the graph.

Let's require the multiplication to be total.

For that to work, every pair of arrows is going to have to go to and from the same
graph node. As a result, there is only one graph node (and there are multiple arrows
from that node to itself).

This gives us a view of monoids as categories. We have identites (every arrow is
an identity, in this view).

However this view isn't super useful, because every arrow is the same. This model
kind of sucks. We could modify how we look at the model to make it more interesting.
Or... we could let the same thing happen by starting somewhere else.

# Monoids (From Semigroups): Identities

Semigroups have associative, total multiplication, but they don't have to have
identities. Our example of a finite semigroup was to start with some arbitrary set
$$A$$. Then we considered a set of functions, $$F$$, where each function $$f \in F$$
is from $$A$$ to $$A$$. Function composition in any set is total and associative.

But the only function which is an identity under composition is the function that
maps any $$a : A$$ to itself, which we write as either $$f(a) = a$$, or $$a \mapsto a$$.
We didn't require that this function be in $$F$$ when we considered semigroups.

For monoids, we're going to require it.

The identity function is a function that "does nothing." This is spectacularly useful.
Not only do we know that there is an identity that works for every function in $$F$$,
we know what it is. And if we need to conjure up an element of our monoid to use somewhere,
we can always safely use this identity because multiplication by it will never "mess up"
another value.

# Monoids (From Unital Magma): Association

Starting from an operation which has an identity and which is total, we can get a monoid
by also requiring it to be associative. I unfortunately don't have a good way to
visualize this change. Please let me know if you do!

Correspondingly, a structure which is both a unital magma _and_ a semigroup will always
be a monoid.

# Monoid Examples

## Booleans

Let's consider some examples. Let's start with the set $$\mathbb B = \{ true, false \}$$. Let's pick
the common operation "or":

$$\begin{aligned}
false &\| false &= false \\
false &\| true  &= true  \\
true  &\| false &= true  \\
true  &\| true  &= true  \\
\end{aligned}$$

As we can see, $$false$$ certainly acts as the identity on either side. Operating false
on false gives false, false on true gives true. We can check that it's associative
(exercise). And, the above table shows that it's total.

This gives us the monoid $$(\mathbb B, false, \|)$$, or "booleans with or."

Sometimes, there's more than one way to imbue a set with a monoid structure. What if
we pick a different operation?

$$\begin{aligned}
false\ & \&\&\ false &= false \\
false\ & \&\&\ true  &= false \\
true\  & \&\&\ false &= false \\
true\  & \&\&\ true  &= true  \\
\end{aligned}$$

Now $$true$$ is an identity[^1]. This gives us a different monoid over the same domain,
$$(\mathbb B, true, \&\&)$$, or "booleans with and."

## Pointed Semigroups

Notice that we can recover a semigroup from any monoid by simply "forgetting" the identity
element. If we neglect to point out that $$true$$ is the identity of $$\&\&$$, we get the
_semigroup_ $$(\mathbb B, \&\&)$$.

There's a way that we can describe functions between _structures_ - the function
from monoids to semigroups, defined by $$F(D, e, *) = (D, *)$$, is known as a _forgetful
functor_. I'll probably have some future posts exploring this idea further, as it can be
combined with category theory to do some pretty general things.

The nicest thing it can do is _be reversed_. If we have some semigroup, we can attach a new
element $$e$$ to the domain of the semigroup and define $$e$$ to be the identity. This new
$$e$$ is called a "point,"[^2] and it might not be distinguishable from every element already
in the domain. If the semigroup already had an identity which was forgotten, then we can prove
that $$e$$ and the identity are one and the same.

Regardless, sometimes we have a semigroup without an identity, and this construction allows us
to summon one out of thin air.

Let's consider a little bit of C code for a moment.

```c
struct SomeSemigroup; // opaque

SomeSemigroup *times(SomeSemigroup x, SomeSemigroup y);
```

By assumption, `times` here is an associative operation, which may or may not have
an identity element. We don't know. But we can _extend_ the operation with a new
identity as follows;

```c
SomeSemigroup *id = NULL;

SomeSemigroup *times_extension(SomeSemigroup *x, SomeSemigroup *y) {
    if (x == id) {
        return y;
    }
    else if (y == id) {
        return x;
    }
    else {
        return times(*x, *y);
    }
}
```

This new semigroup is now a monoid. The identity is `NULL`. It may be that there was
already some other identity in semigroup; in this case, `NULL` is functionally
indistinguishable from that other identity within the semigroup structure.

A common example of a semigroup in programming languages is non-empty array(list)s. We can
append any two arrays to get a third, and this append operation is associative. We have an
additional (useful) structure that we can lookup any element in an array and get something
meaningful back, since they are never empty.

But sometimes, when initializing an array for an algorithm, for example, it's truly useful
to be able to say that it is initially empty. We can recover a structure where that is
allowed by appending a new point to our array semigroup, the null pointer, and extending
the concatenation operation as above.

Now even if the language allowed empty arrays (as most do), the empty array and the null
pointer are functionally indistinguishable. You can't lookup values from either of them,
and appending either one to another array will result in that other array.

We say that the null pointer and the empty array are _isomorphic_. They carry the same
information. In this case, that is no information at all! We can easily write a pair
of functions to witness the isomorphism[^3]. One of them sends the empty array to `NULL`,
the other sends `NULL` to the empty array. While we generally have a _structural_
definition of equality in programs, what we often truly want is _informational_ equality,
where two objects are equal if they are isomorphic.

The exact meaning of isomorphic can differ with respect to context. Sometimes the structure
is information that we care about (for example, trees). Thinking about what exactly equality
means for a particular datatype and implementing it correctly can avoid major headache-
inducing bugs. The concept of monoids (indeed, unital magma) tells us that if we have
two different objects which are _both_ the identities of a type's core operation, then we should
probably be writing a _normalizing_ function, which replaces one with the other, because they
must be functionally indistinguishable.

## Lists

What if we have some unknown type $$T$$, and we want to imbue it with a monoid structure?
Perhaps we want to be able to put off deciding what binary operation to use until later,
if there are several choices. Or perhaps there's no reasonable choice at all, but we
still want to be able to put things together.

One common example of this is validation, or _parsing_. We may encounter an error while
validating a piece of data, but we don't want our program to just fail immediately.

Instead, we want to combine all the errors that we discover during the entire process of
validation (or at least, as far as we can go), so that the user can get as much relevant
information as possible back from our tool. It's not really meaningful to "combine" errors.
We could append the callstacks and append the messages, but that doesn't tell anyone anything.
If anything, it's actively confusing. Instead, we just want some monoid structure that leaves
individual errors alone.

We can always construct such a structure. Instead of considering elements of $$T$$ itself,
we consider elements of a new type, $$L(T)$$. $$L(T)$$ is the type of lists containing
elements of type $$T$$. Each element of $$T$$ corresponds exactly to a singleton list
containing it. We call the function from $$T \to L(T)$$ an _injection_, because it "injects"
elements of $$T$$ into this monoid structure. For example, the definition might be as
simple as `inject(t) = List.singleton(t)`, in a language where this syntax makes sense.

Now the monoid multiplication is list concatenation, which as above is associative. If we
need to conjure up an element from nowhere, we have to be able to do that _without_ relying
on the existence of things in $$T$$. Perhaps we are initializing our parser and we need
some initial value for the set of errors. It's perfectly safe to choose the empty list,
because it is the identity of this monoid and the concatenation with non-empty lists
later won't cause weird things to happen to our structure.

This is a very real example of why being able to conjure up a "nothing" element of a type
is extremely useful!

If you're a programmer, you've probably written loops like this before:

```c
bool flag = false;
while (someCondition) {
    do_some_work();
    flag = flag || checkFlag();
}
if (flag) {
    clean_up();
}
```

`flag` here is acting in the $$(\mathbb B, false, \|)$$ monoid. If the body of the loop
is also implementing some type of monoidal behavior[^4], we could _combine_ these two
monoids and possibly clean up our code.

## Turing Machines

I want to leave off this overview of monoids with a quick proof that every program can be
described by the structure of some monoid.

Firstly, it is well known that any program can be represented by a Turing Machine. It's generally
fairly easy to find such a machine, because the semantics of the programming language are
usually defined in terms of an abstract machine, which is then implemented in terms of some
assembly language, which is itself (usually) quite close to a Turing Machine on its own.

A Turing Machine is its own kind of structure, generally written $$(Q, \Gamma, b, \Sigma,
\delta, q_0, F)$$. In order, we have a set of states, a set of symbols that we can write to
some kind of memory, a designated "blank symbol," which appears in any memory cell that has
not yet been written to, a subset of $$\Gamma$$ representing the symbols which can be in
memory before we start (the "program input"), a set of _transitions_, an initial state,
and a set of final states.

The Turing Machine starts in the "initial configuration," where the memory has some input
symbols in it, the machine is pointed at a particular memory cell, and is in the initial state.
It executes by using the _transitions_, which describe for each state, how to use the value
of memory currently being looked at to decide how to modify the current memory cell and switch
to a different one, as well as possibly also switching states.

In any configuration of the turing machine, we can apply some sequence of transitions to get
to a new configuration. A record of configurations that we move through between the initial
state and a final state is called an _accepting history_.

A Turing Machine accepts an input (computes a result for that input) if and only if an accepting
history exists. An accepting history is a chain of states, all of which are valid. So given any
two states in an accepting history, we can find a (composition of) transition(s) which moves
one to the other.

This structure is monoidal. We have the set $$C$$ of configurations for our machine $$M$$.
We can define, for any input $$w \in L(\Sigma)$$, the function
$$T_w : C \to C$$. $$T_w(c)$$ tells us which state $$c'$$ we will end up in if the "input" in memory
_begins with_ $$w$$; if it does not begin with $$w$$,
then the result is whichever state $$M$$ uses to reject an input.

The set $$T = \{ T_w \ | \ w \in L(\Sigma) \}$$ is the domain of our monoid. The multiplication
is the composition of these transition operators. The identity transition is $$T_\emptyset$$,
because every input can be seen as _beginning with_ the empty string. Any two of these can be
composed, and composition is associative, hence we have a monoid.

This is called the _transition monoid_ of a machine. It formalizes the notion that any computation
can be broken down into steps which are independent of each other, and the results of those steps
can be combined according to some monoidal structure to produce the result of the program.

#### Why?

Why is that useful? After all, the construction is, well, rather obtuse. It does tell us exactly
how to _find_ such a monoid, but that monoid doesn't exactly lend itself to pretty code. An example
of the constructed monoid might look like this, in a dialect of C extended with nested functions
that are safe to return pointers to[^6].

```c
typedef void*(*step)(void*) ConstructedMonoid;

void *id_step(void *state) {
    return state;
}

step *times_step(step *step1, step *step2) {
    step *new_step(void *state) {
        return step2(step1(state))
    }
    return new_step;
}

step *id = id_step;
step *(*times)(step*,step*) = times_step;
```

Here the monoid is the set of all functions which take a pointer and return a pointer. The
pointer is presumed to point to any relevant information that the program needs to continue.
The identity is the function which returns the state unchanged, and the multiplication composes
two steps.

Naively, this is stupid. We don't gain anything by writing our program this way, and really
it just makes the program more opaque. Chances are we were _already_ breaking the problem down
into smaller steps in a more readable fashion.

This idea becomes _useful_ when the monoid is isomorphic to a simpler monoid. In that case,
we can disregard the elements-are-functions notion, and replace the functions with the elements
of the simpler monoid. Each step of our program becomes as simple as generating the next monoid
element, with regards to the globally-available input but _without_ caring about the results
or side effects of other steps. Then we multiply all the resulting monoid elements together
and the result falls out.

We used this pattern when writing an autograder for a course taught in Haskell, which requires
some functions to be a written in a certain recursive style. The grader is implemented as a
a function converts a stack of nested function definitions into a flattened list of bindings
along with all the other names in scope. This transformation is implemented by (recusively)
translating each individual definition into an element of a monoid and then multiplying them
together as we recurse back out. Each element of the flattened list of bindings is then checked
_individually_ for violations of the desired property and the results are monoidally combined
into the final result.

#### Ability to Generalize

Any monoid corresponding to a turing machine in this way ends up being isomorphic to another
monoid which is an instance of a special kind of monoid called a _monad_. Monads are studied
in category theory, which we're not going to get into here, but some programming languages
make them explicitly representable. This makes it possible to express programs by describing
how to translate inputs into elements of the monoid and combining them together.

A monoid in this way represents the structure of the programming environment. What that means
is that it represents what side effects can be performed by the functions which return
elements of the monoid - throwing errors, manipulating a shared state, things like that.

By generalizing such a program from a specific monoid to "any monoid which contains that
monoid as a substructure," we can generalize the environments in which a function will work.
Rather than having a function which works "with my database," we can have a function which
works "with _some_ database." We can then easily test our programs with a mock database,
while using a real database for our actual business environment. Since it's the same code
running in both cases, we can be confident that the tests passing means our logic is sound,
even though the underlying database is different.

This is related to other programming patterns like dependency injection. There's a very
rich space of programming constructs and patterns that can be explored here to find
ways to write cleaner programs. Haskell has a collection of libraries known as
_algebraic effect libraries_ which provide implementations of this idea.

We'll explore this concept further in future posts, I hope, but now back to the math.

# Groups

A group is a monoid with inverses. Alternatively, it is a total groupoid, or an associative
loop.

A monoid $$(D, e, *)$$ is a group if for any $$x \in D$$, there is an element $$x^{-1} \in D$$
such that $$x * x^{-1} = e$$. We can easily prove that inverses are unique, which is a good
exercise.

Many, many things that we say in day-to-day mathematics form groups. For example, the monoid
$$(\mathbb Z, 0, +)$$ is also a group, because for any $$x \in \mathbb Z$$, we have
$$-x \in \mathbb Z$$, and $$x + (-x) = 0$$.

However, the monoid $$(\mathbb N, 0, +)$$ is _not_ a group, because negative numbers aren't
in $$\mathbb N$$. The monoid $$(\mathbb Z, 1, *)$$ is also not a group, because for example,
there's no inverse of 2. There's a mechanism by which we could extend this monoid into a group,
and the result would be $$\mathbb Q$$. Perhaps in the future I'll explore this construction in
another series of posts, which would probably build up to how we can define $$\mathbb R$$
_algebraicly_.[^7]

As discussed in the last post, Rubik's Cubes are associated with a group where every
element of the group is a sequence of moves. We can get this group by starting with
all of the 90 degree clockwise rotations of a single face (there are 6 such moves)
and then "closing" the set under concatenation: add to the set every move or sequence of moves
which can be obtained by concatenating two (sequences of) moves in the set. Repeat until
nothing new gets added.

We can check that inverses exist in this group. Call the clockwise rotation of the front face
$$F$$. We can check that $$F^{-1} = F * F * F$$, which we can also write $$F^3$$. This means
that $$F$$ can be undone by performing $$F$$ three more times; $$F^{-1} * F = F^4 = e$$.
The groupoid properties then tell us that (since products are always defined in a group),
if we have the sequence of moves $$A * B$$, then $$(A * B)^{-1}$$ is nothing other than
$$B^{-1} * A^{-1}$$. This corresponds to undoing sequence $$AB$$ by first undoing $$B$$,
and then undoing $$A$$. That's exactly what we expect!

From here, we could develop a rich theory of groups, called _group theory_, and apply it
to the study of a huge variety of real groups that appear in mathematics. Eventually, I'd
like to develop some of that theory in a sequence of posts, and build up to an understandable
proof of the fact that quintic polynomials are not solvable in general in terms of addition,
multiplication, and $$n$$th roots.

# One Final Property: Getting To Work

One other property that we frequently ask of binary operations is that they _commute_. That
is, they satisfy the restriction that $$x * y = y * x$$.

If we have a monoid with this property, we call it a _commutative monoid_. If we have a group
with this property, we call it an _abelian group_. Most of the examples we've discussed so
far are commutative, including the boolean monoids, and the additive group. The list monoid
is _not_ commutative, because order matters in lists. In contrast, the _set_ monoid is commutative,
because sets are unordered. The transition monoids of state machines are not commutative.

If you have a Rubik's Cube handy, check that the Rubik's cube group is not abelian. We could
loosen our restriction a little bit, and ask the following. Given some group $$(G, e, *)$$,
what is the largest subset $$C \subseteq G$$ such that for any $$z \in C, g \in G$$, we have
that $$z * g = g * z$$? We say that every element of $$C$$ _commutes_ with every element of $$G$$.

We can prove that this set $$C$$, called the _center_ of $$G$$, is itself a group:

1. $$C$$ has an identity element, because for any $$g \in G$$, $$e * g = g * e$$, so $$e \in C$$.
2. The restriction of $$*$$ from $$G$$ to $$C$$, $$*_C : C \times C \to C$$, is total. Consider two elements
   $$y,z \in C$$ and some $$g \in G$$. Then we have that
   
$$\begin{aligned}
(y * z) * g &= y * (z * g) & \text{Associative} \\
&= y * (g * z) & z \in C \\
&= (g * z) * y & y \in C \\
&= g * (z * y) & \text{Associative} \\
&= g * (y * z) & y,z \in C\\
\end{aligned}$$

so $$y * z$$ is also in $$C$$.

3. If $$z \in C$$, then $$z^{-1} \in C$$ as well:

$$\begin{aligned}
z^{-1} * g &= (z^{-1} * g) * e & \text{Identity} \\
&= (z^{-1} * g) * (z * z^{-1}) & \text{Product of Inverse} \\
&= z^{-1} * (g * z) * z^{-1}   & \text{Association} \\
&= z^{-1} * (z * g) * z^{-1}   & z \in C \\
&= (z^{-1} * z) * (g * z^{-1}) & \text{Association} \\
&= e * (g * z^{-1})            & \text{Product of Inverse} \\
&= g * z^{-1}                  & \text{Identity} \\
\end{aligned}$$

Since $$C$$ is a subset of $$G$$, and $$(C, e, *)$$ is still a group, we call $$(C, e, *)$$ a
_subgroup_ of $$(G, e, *)$$ and we write $$C < G$$.

The center of a group is nice because it is the largest subgroup which is abelian. In some
groups, the center is trivial, meaning it contains only the identity.

I'll end this post with something that I consider to be a good thought exercise: is the
center of the Rubik's Cube group trivial? If not, how big is it?

By developing some group theory, we could fairly easily prove an answer to this question.

# Conclusion

In this series of posts, we introduced the general concepts of each of the major _single-sorted,
binary-operator_ algebraic structures.  We saw how they arise from adding increasingly stringent
constraints to the binary operator, and attached some examples to each of the weird names.

I hope that abstract algebra feels approachable from here. We can use it to achieve general
results which we can then apply to specific scenarios to get results for free. A further treatment
of abstract algebra would explore some more important structures, notably _rings_, _fields_,
and _vector fields_. These structures are nothing more than adding some new requirements to our
set.

We can improve programs, by implementing the results as very general functions and then
applying the general functions to specific scenarios, achieving great degrees of code re-use
and understandability. Simply by seeing a reference to a common general function, we can
immediately understand the broad strokes of what the function is doing, even if we don't yet
understand all the specifics of the structure we're applying it on.

Most likely, my next posts will be about computer hardware, specifically superscalar out-of-order
processing and how we can use the visual digital logic simulator [Turing Complete](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwj_4eXQuoP5AhVohIkEHeC1DwEQFnoECBMQAQ&url=https%3A%2F%2Fstore.steampowered.com%2Fapp%2F1444480%2FTuring_Complete%2F&usg=AOvVaw3zW9DEhcM5CdezAukGNxTo)
to explore the different components of such systems and how they work together.

Further down the line, future posts will explore some more group theory, and some more direct
application of abstract algebra to programming in Haskell. I want to explore so-called
_free structures_, and how we can use them to describe computations that build other
computations, a technique called _higher-order programming_. We've seen some free structures
in these posts already, though I didn't explicitly call them out[^8]. I also alluded to higher-order
programming via _effects_ earlier in this post, but there are other neat things we can do
with the technique that don't seem to get much exposure.

---

[^1]: Remember from the last post, we proved that identites are unique in unital magma.
      Since monoids _are_ unital magma, identities are unique in monoids too. So 
      **without checking this fact**, we immediately know that $$true$$ is _the only_
      identity. For the remainder of the post, I'll say "the identity" of a monoid, instead
      of "an identity."

[^2]: In the original post in this overview series, I mentioned that distinguished elements
      of structures are always called points, and structures that have any are called "pointed."

[^3]: As discussed in [Setting the Stage]({{ page.previous_in_category.previous_in_category.url }}#sets-setting-the-stage)

[^4]: We'll see in a moment that it always is.

[^5]: Here we use a model of Turing Machines that don't crash. Instead of crashing, they
      have to reject the input. For simplicity, we'll assume only one state causes rejection;
      it's easy to translate a Turing machine with multiple reject states into a machine with
      only one.

[^6]: C++ has such a construct, but I very strongly dislike C++, so this is what we get.
      In a later post, we will review these concepts through the lens of Haskell, where this
      structure will turn out to be... quite familiar.

[^7]: That is, in terms of the properties that the set should have. What might those
      properties even be, and how can we construct a set that has those properties if it is necessarily
      uncoutably infinite?

[^8]: Specifically, our example of magma was the free magma on binary tree nodes, linked lists
      (the semigroups that we got by making the magma associative) are the free semigroups on
      linked list nodes, and general lists containing things of type $$T$$ are the free monoids
      on $$T$$.

      In a deep sense, the composition of functions that we saw when expressing programs as
      monoids described how to take a list of steps to perform and flatten it into a single
      long step that composes all the elements of the list. By generalizing over more interesting
      composition operators (which tend to arise from the "more interesting" monoids that
      we may be lucky enough to be isomorphic to), we can again arrive at a free monoid, this time
      over programs themselves. These free monoids are called _free monads_ and form the basis
      for programming via effects.
