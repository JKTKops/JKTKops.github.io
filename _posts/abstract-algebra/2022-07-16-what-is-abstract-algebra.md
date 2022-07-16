---
layout: post
title: "What is Abstract Algebra, Anyway?"
date: 2022-07-16
categories: math algebra
---

"Abstract Algebra" was a term that I frequently heard growing up in reference to difficult college-level math. When people want to make math sound scary, it's a common go-to. I think this is just because it has the word "abstract" in the name. Certainly, no one who tried to scare me with it when I was young could actually tell me what it was.

Is that because it truly is scary, or simply because they didn't have access to good resources to learn it? We're going to find out!

In this mini-series I hope to approach Abstract Algebra from a simple angle, in terms of motivating examples and building up desirable properties. There will be little-to-no advanced math here. Equational reasoning skills will help, as may some previous exposure to proofs. This post is aimed at hobbyist or experienced programmers with a less formal mathematics background, or at interested pre-university mathematicians. Many motivating examples will be in terms of programming.

This long introductory post aims to introduce the general concept of abstract algebra and motivate why we would want to use it.

With all that out of the way, let's get into the meat of it!

## Why Abstract Algebra?

Indeed, why _math_? Mathematics gives us the tools to talk about various patterns we encounter every day. Algebra (the non-abstract kind) gives us the tools to talk about everyday constraints, like "What time do I have to leave my house to be at work at 9:00?" It can also talk about much more complicated, but still everyday problems, related to money, logistics, cooking, and just about anywhere else you might encounter an equals sign.

Arithmetic gives us the tools to talk about various types of counting. For example, I know I have 12 cookies and 3 friends, and I want to count how many cookies each friend can get.

Calculus gives us the tools to talk about rates and totals. How much does the circumference of a circle increase if you change the radius? What happens if you total up the lengths of the circumferences for circles of various radii?

Geometry lets us talk about shapes without (necessarily) worrying about manipulating numbers.

All of these things let us talk about concrete things, which we can touch and move around, or at least construct physical examples. Abstract algebra is tricky to view this way, because it is _abstract_ by nature. Abstract algebra gives us the tools to talk about _structure_ - what happens when we enforce certain rules on mathematical objects.

But what does that even mean? Consider the natural numbers, $$\mathbb N$$. If we don't give them _structure_, we have a collection of weird symbols like $$3$$ and $$42$$ that we can't manipulate. But we can give them a structure by declaring that they have an _order_. $$2$$ comes after $$1$$, $$3$$ comes after $$2$$, etc. Is there something special about this structure in particular? No, there isn't. We could easily have put a different number first, for example. Or we could use completely different symbols, like emoji. Or why even stop at symbols - a stack of pennies has the same structure!

So what we _really_ gain from abstract algebra is the language to talk about three things:

1. Specific structures, like "the structure of $$\mathbb N$$"
2. Types of structures, such as "structures which can model counting"
3. Relationships between structures, such as how two seemingly different structures (like $$\mathbb N$$ and a stack of pennies) are actually one and the same.

The last one is where I find the real beauty in abstract algebra, and it lets us tie all sorts of different types of math together.

## Let's Get Into It: What is a Structure?

Now that we know what we're talking about at a high level, let's get a bit more specific. Obviously, as programmers or as mathematicians, we see these types of structures all the time. Programmers will recognize the stack of pennies as a linked list (or perhaps some other kind of list). But we can also talk about other shapes of structures entirely, so let's get a bit more precise.

Before we can talk about something have structure, we need to have, well, something. In our running example, the "something" is either $$\mathbb N$$ or a collection of pennies. In either case, we have a collection of things - a _set_.

Every "algebraic structure" is going to start with a set. This set is called the _domain_ of the structure, or the _carrier set_. There are structures that have more than one domain, but we're not going to talk about those in this post. In this post, we will only discuss _single-domain_ structures. 

Then, we put structure on the set by defining (arbitrarily!) a _relationship_ between the objects in the set. For our "natural number structure," that relationship is the _followed-by_ relation. $$2$$ is _followed by_ $$1$$, etc. Now we ask ourselves what properties this structure has. We could list a few:

1. Every natural number is _followed by_ another natural number.
2. $$0$$ is special - there is no natural number $$n$$ for which $$n$$ is _followed by_ $$0$$.
3. For every natural number which is _not_ $$0$$, there is exactly one other natural number which it follows.

In fact, these 3 properties exactly describe our structure. Here, we're using our language in way #1 from above - we're talking about a specific structure.

Since these 3 properties exactly describe our structure, we should be able to state the same 3 properties for stacks of pennies:

1. For any stack of pennies, we can add another penny on top to get a taller stack.
2. The empty stack of pennies is special - there is no stack of pennies which we can add a penny to and end up with the empty stack.
3. For every stack of pennies which is not empty, there is exactly one stack of pennies which we can add a penny to get this one.

These are the same 3 rules, just stated with slightly different wording. And they describe _exactly_ the same structure, merely with a different carrier set.

That means that anything we prove about stacks of pennies immediately applies to natural numbers as well - or to _anything else with this structure_. Thus what really matters is the properties that define our structure, and not the structure itself. Let's rephrase our properties in a more general way.

1. For any element $$x$$ of the domain, there is an element $$\sigma(x)$$ which is also in the domain.
2. There is a special element $$0$$ in the domain, and there is no $$x$$ such that $$\sigma(x) = 0$$.
3. For any element $$x \neq 0$$ in the domain, there is exactly one element $$y$$ in the domain such that $$\sigma(y) = x$$.

Now we refer to some generic "domain," but we don't actually care what it is. We only care that it has the structure dictated by these properties. These properties are commonly known as the "Peano axioms"[^1].

This structure is generically called the natural numbers, whether we use $$\mathbb N$$ as the carrier set or stacks of pennies. I want to emphasize that our conceptual understanding of the structure comes from the properties, and not from the carrier set.

The technical designation for this structure comes from the following facts about it:

1. The operation that dictates the structure, $$\sigma$$ is _unary_, or takes one argument.
2. The operation distinguishes a particular element in the set, $$0$$. Mathematicians call such operations "pointed."

So we would call the natural numbers a "pointed unary system."

## If the Dress Fits, Model It!

We've now taken a step _away_ from concrete objects, and described how we talk about two things having the same structure. We define the structure's properties, and show that both things satisfy those properties. We call these properties the _axioms_ of the structure. When something concrete satisfies the axioms of a structure, we call it a _model_ of that structure. It's also pretty common to just call it the structure itself, so the set of pennies along with the "add a penny to the stack" operation may just be called the "penny system," even though it's the same system as the natural numbers.

Whenever we prove something generic about the axioms, we've also proved that thing about _every_ model of the axioms. Let's see an example.

The Peano Axioms we've been looking at describe _counting_. $$\sigma$$, conceptually, is "count to the next number." It turns out that anything with this structure also has a much more interesting structure. Using _counting_, we can describe _arithmetic_.

Let's propose some additional properties for addition:

1. For any two values $$x,y$$, $$x + y$$ exists.
2. If $$x + y = z$$, then $$y + x = z$$ as well.
3. If $$(w + x) + y = z$$, then $$w + (x + y) = z$$ as well.

Let's suppose we have any model $$(D, \sigma)$$ of the Peano axioms. This notation means that the carrier set is $$D$$, along with the operation $$\sigma$$ for counting. We can define addition as follows:

$$\begin{align} a + 0 &= a, \\ a + \sigma(b) &= \sigma(a + b). \end{align}$$

Now we want to check that it satisfies the properties we asked for. First, let's suppose $$x,y \in D$$. One of the two patterns above will fit $$x + y$$, so we definitely have a definition for $$x + y$$. Since (by axiom 1) $$\sigma(a + b)$$ will always again be a natural number, we have that $$x + y$$ does really exist.

What about the second rule, which is usually called _commutivity_?

First, let's show that $$a + 0 = 0 + a$$. If $$a = 0$$, we're done. So let's assume that $$a \neq 0$$.

$$\begin{aligned} 0 + a &= 0 + \sigma(b) & \text{By axiom 3} \\
&= \sigma(0 + b) & \text{By the second pattern} \\
&= \sigma(b + 0) & \text{Explained below} \\
&= \sigma(b) & \text{By the first pattern} \\
&= a & \text{By definition of } b\\
&= a + 0 & \text{By the first pattern, applied in reverse} \end{aligned}$$

What about that step in the middle, where we used the fact that $$\sigma(0 + b) = \sigma(b + 0)$$? Doesn't that rely on what we're trying to prove?
Well it does - but $$b$$ is closer to $$0$$ in the $$\sigma$$-chain than $$a$$ is. So we could "unroll" the proof by writing out the steps for $$b$$, and for $$b-1$$, until we eventually terminate at $$0$$. We know that we will eventually terminate, because our Axiom 3, or the _uniqueness axiom_, tells us that eventually, we will have to reach 0, and we know that our claim holds for 0. Formally, this reasoning is called _induction_.[^2]

To prove the general claim of $$x + y = y + x$$, we would use a similar inductive step to show that $$x + \sigma(y) = \sigma(x) + y$$, and then we would be able to "move" layers of $$\sigma$$ from one side to the other until we get the balance correct. The details are left as an exercise.

I'll also leave proving the 3rd property of addition, _associativity_, as an exercise. It's less annoying than commutativity.

## So what?

So we've shown that any system that supports _counting_ also supports _adding_. Why is that important?

Addition is a _binary_ operator. It takes two arguments, both natural numbers in this case, and produces a third natural number. Addition puts a much richer structure on the domain. Instead of only being able to say that $$3 \to 4$$, $$4 \to 5$$, and $$5 \to 6$$, we can now make much broader, more comprehensive statements like $$(3, 4) \to 7$$. And this doesn't only apply to numbers, but to _any_ set that it makes sense to "count." That's a very powerful result!

Some of the richest - and most practical - structures arise from single-domain, binary-operator systems. Mathematicians define a whole heirarchy of such structures, starting with no properties at all (a set with no operation) and gradually increasing the requirements of the operation to become more and more specific. The significant parts of this heirarchy look like this.


<img src="/assets/abstract-algebra/heirarchy.svg">

The structure we've been talking about here is a commutative monoid. As we continue to develop the language of abstract algebra, we will learn conceptually exactly what that means.

Hopefully I've piqued your interest! In the next few posts we will describe each type of structure in the diagram and the arrows that it points to. That is, I want to describe each structure in terms of what it _adds_ to something that we are already familiar with. As we get further down the line, we will see how identifying these types of structures can improve programs, clarify ideas, and help us make deep observations about the nature of (mathematical) existence.

-----

[^1]: For simplicity, I've combined the fact that $$0$$ exists with its special property. But technically these are two separate properties. If we don't dictate that $$0$$ must exist, then the empty set would satisfy all of the other properties! So the Peano axioms actually have _4_ properties, with the first one being "0 is a natural number."

[^2]: Typically, induction would be stated as a Peano Axiom. But the typical statements are all very difficult to parse and not conceptually enlightening. However, my handwavy description of "unrolling" the proof can be formalized. It turns out that defining that 0 is the "the smallest natural number," in the way that we did, is equivalent to giving induction as an axiom. The details of the proof are outside the scope of this post.
