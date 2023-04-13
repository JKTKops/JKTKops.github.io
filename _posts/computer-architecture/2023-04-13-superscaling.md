---
layout: post
title: "Superscaling"
date: 2023-04-13 00:00:00 -4
categories: computer-science hardware
---

Continuing with the computer architecture posts, this time we're going to explore superscaling.

Before we start, my goal here is to eventually describe _in detail_ the various components of a
modern CPU architecture and how they work (and how they work _together_).
I find that it's quite difficult to find resources online
about how such things actually work, and I think I can fill that gap.
Therefore, this exploration is mostly setting the stage - exploring what those components are, but not how they work.

There are some other great sources that cover modern architecture
principles at a high level. [This one](https://www.lighterra.com/papers/modernmicroprocessors/) is pretty good, for example.
It's not a prerequisite though. This series is self-contained.

* TOC
{: toc}

# The Goal - Go Fast

The driving goal of most computer architecture innovations is
to go fast[^1]. In the [last post]({{ page.previous_in_category.url | relative }}),
we saw how _pipelining_ lets us speed up the processor
by separating out the steps of an instruction, and executing
one instruction in each step on every cycle. In contrast,
a simple computer model executes one instruction every cycle.
With pipelining, every cycle we make a little bit of progress on several different instructions.
These different instructions are independent[^2] and so we can
make these little bits of progress (almost) completely in parallel. As a result, we need less time
per cycle to make progress. So we still finish one
instruction per cycle, but the cycles are a lot shorter.

So why not just make our pipeline extremely deep? For example,
if we put a pipeline fence after every logic gate, we could run our clock extremely fast.
However, above, we assumed that each instruction in the pipe was independent.
In the last post, we saw that small pipelines can sometimes
even justify that assumption. But if we make the pipeline
very deep, that assumption is pretty much guaranteed to break.
So our clock will be very fast, but we won't be able to keep the
pipeline saturated with instructions. Then we wouldn't
be finishing one instruction per cycle, and we wouldn't be going fast.

We want to go fast.

There's another problem too, which is that there's some
overhead for each pipeline stage. The stages have to be separated
by pipeline registers, which are their own bit of circuitry and
add some more propagation delay to the overall design. 
If we put too many of them, the overhead from the pipeline registers
starts slowing us down more than splitting the stages can speed us up.

So making the pipeline as deep as physically possible is not the answer. What are some other potential answers?

1. Make the circuit smaller. Smaller _physical_ distance for signals to travel means they will not have to travel as long.
2. Try very hard to _find_ parallelism in the program and exploit it.
3. Demand that users write better programs.

Idea #1 is just a plain win - if we can make the circuit smaller, we should. One of the first microprocessors, the Intel 4004, had transistors about 10 μm across. Recently, mass-production of designs using transistors about 20 _nm_ across has begun.[^3]
We're starting to run up against the physical limit of how small something can be, but until then, we can keep trying to eek out more speed by getting smaller.

Idea #2 is what we were getting at with pipelining. As long as that independence assumption holds, we can find
_instruction-level parallelism_ in the program and execute several
instructions in parallel.

Idea #3 sounds stupid, but in fact, is a very good idea. We shouldn't let the idea that we have to be
fast in _every possible situation_ prevent us from
doing something that is fast only for programs that
know how to take advantage.

There's a balance that we can achieve between #2 and #3,
where we try to be fast in general, but what we choose to do works best for programs
that are written to work with what we're doing.
And this actually works in practice, because it's not _people_
writing the programs! Compilers write the programs, and compilers are very good
at emitting assembly that can work well with whatever constraints we put on "good programs."

But even in the best case, there will be lots of opportunities for instruction-level parallelization that are only apaprent at runtime.
For example, across iterations of a loop.
Even the smartest compiler can't help with that.

So idea #3 can get us surprisingly far, but it has to go _together_ with idea #2 if we really want improvements. Trying harder to find parallelism results
in hardware that has to work very hard, which has colloquially become known
as the "brainiac" paradigm: going faster by making processors smarter.

So with that long introduction out of the way, let's look at another way to exploit the parallelism we find.

# Superscaling

Ah, the title of the post!

Superscaling is a technique to execute multiple instructions at a time, in a _different_ way from pipelining.

Remember that basic pipelining lets us complete at most 1 instruction per cycle. That still puts quite a limit on our speed. We're executing multiple instructions at once, by separating them into stages.

What if we could straight-up have multiple instructions _in the same stage_ at the same time? Enter superscaling.

So-called "superscalar architectures" are architectures that can have multiple instructions in the same pipeline stage at the same time. As long as the instructions are independent, everything still works great! We do, however, consume a lot more _space_, after all, we need circuitry for multiple instructions now.

As we'll see in future posts, some of that growth is non-linear.
Some techniques need to analyze every pair of instructions
that passes through a stage, every cycle. This means that we need quadratic circuit size
in the number of instructions we can handle at once (the "width" of the architecture).
As a result, modern processors are typically between 4 to 6-wide superscalar architectures.

Let's look into how this works in more detail, using an example based on the Intel P5 microarchitecture.

## Two Separate Pipelines

The P5 microarchitecture has two separate pipelines called the _U_ pipe and the _V_ pipe. Here's a high-level block diagram of the integer datapath:[^4]

<img src="/assets/architecture/p5-high-level.svg">

In a sequential view, the instruction in the U pipe is the first instruction,
and the instruction in the V pipe is the second instruction.

If these instructions could have complicated interactions with each other,
we would have to build complicated hardware to resolve those interactions.
That could be bad - it could be slow or just too power-hungry.
To save on complexity, there are a whole host of
restrictions on "pairing."

The first decoding stage has to decode enough of two  instructions at a time to figure out if _pairing_ is possible.
If it is, then the two instructions are paired and
sent to the corresponding pipes at the same time.

The U pipe can execute any instruction, so when pairing
is not possible, the first instruction is sent to the U
pipe and the other instruction has to wait.

These restrictions come
back to our idea #3 above. It's on the user to write a program
such that adjacent instructions are pairable. The processor
will still work if they aren't - but it will work up to twice as fast if they are.

## New Perspectives

Even with pipelining, it's been relatively easy to view out processor as executing the
code in exactly the order specified. As we explore more
techniques, this will become more and more difficult.

Already, it can be tricky. When two instructions execute at once, we have to think about new kinds of issues that simply cannot happen in a scalar[^5] architecture, even a pipelined one.

I suggest keeping this in mind. Even when it seems obvious that something "should work,"
question it. Very strange things can go wrong if we're not careful!

## Hazards Revisited

In the last post, we looked at how RAW hazards, or Read After Write hazards, can complicate the implementation of a pipeline and require forwarding.

Now that we can execute more than one instruction _at the same time_, some new types of hazards are possible.

I'll demonstrate by using some actual x86 code here, but no worries. We can demonstrate all of these new issues with simple instructions:

* `mov r1,r2` copies the value in `r2` to `r1`
* `j <label>` unconditionally jumps to the label.

### RAW Hazards

First, let's consider the same RAW hazard as with basic pipelining, using a chain of `mov` instructions:

```x86asm
  mov bx, ax
  mov cx, bx
```

There's a RAW hazard between these instructions, since the second one Reads `bx`
After the first one Writes `bx` (caps for emphasis).

In the simple pipeline, this wasn't a huge deal; we could easily forward the new value of `bx` from the writeback stage to the execute stage by the time it was needed.

Here we have a new kind of problem - the V pipe needs `bx` _at the same time_ that the U pipe produces it. 
In theory, this could force the V pipe to stall while the U pipe proceeds,
which would cause our pairings to get desynchronized.

We have no choice but to prevent this from happening entirely.
If two instructions have a RAW hazard,
they can't be paired.

If they aren't paired, then we can use the same forwarding techniques as the simple pipeline.

But wait - question everything! Can we really?

### WAW Hazards

Consider this sequence:

```x86asm
  mov cx, ax
  mov cx, bx
  mov dx, cx
```

The first two `mov` instructions have no RAW hazard, so we can pair them. But they _both_ write `cx`, so what happens when it's time to writeback?

Our writeback stage now needs to be able to detect these conflicts and resolve them. The resolution is simple - the instruction in the V pipe, which is logically[^6] second, wins.

Our forwarding datapath also needs to be able to detect such conflicts and
determine not just if _any_ value should be forwarded,
but _which_ value.

This is one of those quadratic scaling issues I mentioned earlier,
because any pair of instructions might have a WAW hazard.

Theoretically, this isn't a huge deal. However, the P5 chose to resolve this issue
by pre-empting it - WAW Hazards also prevent pairing.

### WAR Hazards

The last type of data hazard is called a WAR Hazard, or write-after-read. These are also known as "false hazards," because they don't matter unless we start doing some pretty wacky things.

The things we're doing here aren't wacky enough.

Yay for us, that means we don't have to worry about these ones.
We'll revist WAR Hazards in the next post on the _scoreboard_ technique.

### Control Hazards

In simple pipelining, we saw _control hazards_ when instructions are able to enter the pipeline after a branch, but before that branch completes execution.

Such instructions could end up modifying the state of the machine if we weren't careful.

The solution previously was to "squash" them.
Once a branch completed execution, we could squash
the pipeline up to whichever stage the branch was in.
This replaces all of the instructions that shouldn't
have slipped in with `nop`s.

Now things are more complicated:

* What if _both_ instructions being executed are branches? 
* What if the U-pipe instruction is a branch, and the V-pipe instruction writes memory?

Let's look at the first case:

```
  j A
  j B
```

If we allow these to pair, they will both enter the execute stage at the same time. Which one wins?

For WAW hazards, the answer was that the V-pipe instruction wins.
But here, we need the U-pipe instruction to win!
Once again, these differences can complicate hardware.

What about the second case?

```
  j A
  mov [ax],bx
```

That `mov` instruction is a bit different from the other ones we've seen.
The `[ax]` syntax means "the value **in memory** at the address given by `ax`."

In this case, the branch and the memory write would execute at the same time.
By the time we find out that there's a branch to take (or equivalently, that we've mispredicted the branch), 
it's too late to stop the memory write.

This is bad.

In fact, this is so bad that the P5 architecture 
doesn't allow paired branches in the U-pipe _at all_.

That prevents both of these weird issues. It prevents the first issue,
a U-pipe branch being paired with a V-pipe branch.
And it prevents the second issue, a U-pipe branch being paired with a memory write.

This means that branches can only pair in the V-pipe, which is a bit different from the typical case.
Since the U-pipe can execute any instruction 
(including branches - they just can't be paired)
the typical case is that the V-pipe instruction prevents pairing.

Doing things this way allows instructions to pair with branches without running the risk of hitting the memory write issue.
Since branches are common in practice, making them completely
unpairable would be catastrophic.

### Structural Hazards

This is a completely new type of hazard, which we will see a lot more of in the next post.

Various instructions need access to various _hardware resources_ in order to do their jobs.
For example, an add instruction needs access to an adder.
Load and store instructions need access to the cache.

Previously, when only one instruction could be in a stage at a time,
we never had to worry about resources availability.
Now we do.

Designing a cache that can service multiple accesses
on the same cycle is extremely difficult.
It's worth it - modern caches can typically support 2 loads and 2 stores at the same time.
Cutting-edge caches can support more.

The P5 microarchitecture is older and couldn't pay the cost
of such a cache. As a result, if paired instructions both need
access to the cache, it results in stalls.
However, we still allow them to pair.
The parts of the instructions (if any) that don't need
to access the cache can still execute in parallel,
so the overall cycle count will still be lower than
if we refuse to pair them.

I'd show a timing diagram here, but I haven't been
able to find a way to format a 2-pipe diagram that
makes any sense. If you know of a format, let me know!

Any other kind of shared resource is also liable to cause
structural hazards.

Resolving a structural hazard requires deciding which
request for the resource will happen first, a process called _arbitration_.
We then have to allow the selected request to access the resource, while stalling the other request.
We have to keep track of pending requests so that we
don't forget them, which would be catastrophic.

## Working Harder in Common Cases

A theme of all of the techniques we will see in the future is that there are some
cases where doing better is _possible_, but simply
too expensive.
We've just seen a couple involving hazards!

There's a particular very common case of RAW/WAW hazards in x86. x86 has `push` and `pop`
instructions that use a dedicated _stack pointer_
register to help the programmer manage the call stack.

A function might start with a sequence like this:

```
  push  bp
  mov   bp,sp
  push  ax
  push  bx
```

This sequence sets up the stack frame for a function.
`bp` is used for the frame pointer.
Since code like this is involved in every function call,
this is an _extremely_ common pattern.

But `push` both reads _and_ writes `sp`, the stack pointer register.
That means every single `push` instruction in this code causes a hazard!

This case is so common that it's worth trying to do better.
The P5 microarchitecture contains "`sp` predictors"
that recognize `push`/`pop`/`call`/`ret` instructions
(all of which use `sp` on x86) and compute the `sp`
value that the V-pipe instruction should see.
They can do this in a pipeline stage _before_ the U-pipe instruction executes,
so that there is no delay needed for `sp`!
In particular, these calculations happen in the Decode/AG stage,
which is responsible for all types of Address Generation.

The term "predictor" here just means that they compute
something in advance. They aren't like branch predictors,
which can be wrong. The `sp` predictors always produce the right value.

These predictors are able to break hazards on `sp` caused by those 4 instructions,
which enables those 4 instructions to pair.[^7]

## The EFLAGS Register

Those of you familiar with x86 might have noticed something worrying -
almost always, the instruction immediately before a branch
computes a condition for the branch. x86 conditions
come in several forms, and the computed conditions
are stored in an _implicit_ register called the `EFLAGS` register.

So, wouldn't such patterns cause a RAW Hazard on the `EFLAGS` register?

Most of the reason for separating register read, execute, and register write stages in the pipeline
is that we have to do complex execution with the
values we read, which takes time; and we need to be able
to forward results quickly, which also takes time,
so it helps if the values to forward come from the
start of a stage instead of the end.

However, access to `EFLAGS` is simple.
Each instruction can only read it _or_ write it.
Since it's not general-purpose like the other registers,
it's also less complicated to keep track of.

This means we can keep the `EFLAGS` register entirely in the execute stage,
and resolve hazards on the register inside the stage!

As a result, `EFLAGS` hazards never prevent pairing.
The hardware to resolve those hazards inside the stage
adds some complication, but it's not too bad,
and it is _absolutely_ worth it.

Something important to point out is that resolving those hazards can introduce delay into the system.
If the U-pipe instruction can't produce flag values
for a long time, and the V-pipe instruction is a branch
that needs to read them, the branch circuitry has to
wait for the flag values to propagate through the stage's
circuit.
The fact that this is possible at all would slow down our clock
even when the instructions in the execute stage don't care.

We have to design our pipeline carefully so that this extra delay is not a limiting factor.
In the P5 microarchitecture, the pipeline stages are split up
so that the execute stage instructions can produce their
flag values very quickly. Most of the stage's delay
comes from the fact that it also handles memory writes.
If things would take too long, the execute stage can
stall. This allows us to trade extra cycles in some rare cases
for clock speed in _all_ cases. That's a good deal!

# Conclusions

Multi-pipe superscaling is the first technique we've seen that lets us
execute multiple instructions per cycle. For the first
time, we've been talking about _instructions per cycle_ instead of _cycles per instruction_.

We're still limited by slow instructions. Multiplication typically takes several cycles, for example, which will stall _both_ pipes until its done.

We saw a new type of hazard that was not a problem before.
We also saw how it can be worth it to try extra hard in a common case,
even when it's too expensive to resolve a complication in general.
This phenomenon will continue through every other technique we see.

The P5 microarchitecture was far from the first superscalar architecture,
but it was the first superscalar architecture for x86.
Due to x86's complexity, such a thing was previously
thought to be impossible. In fact, even just being able
to effectively _pipeline_ an x86 architecture was thought
to be impossible!

I chose to use the P5 architecture as an example here because an overview at this level doesn't care
so much about the specific machine language.
But the P5 microarchitecture tells a compelling story
in the history of computer architecture.

No matter how complicated the domain gets, no matter
what issues and hazards arise, nothing will stop us
in our pursuit of Going Fast.

Up until this point, we've been constrained by the simple notion
that we read instructions in some order and they should execute in that order.

In the next post, we're going to start to see how even something as powerful as ordering constraints
is unable to stop us from Going Fast.

See you then!

-----------------------

[^1]: The rest are inspired by trying to reduce power consumption. Going fast has historically been the driving factor; reducing power consumption of a technique for going fast comes _after_ inventing the technique in the first place.

[^2]: When they aren't independent, we start running into _data hazards_, described in the [last post]({{ page.previous_in_category.url | relative }}).

[^3]: You may have seen reference to the "3 nm process." This is a misnomer - it's just a marketing term. The transistors produced by the 3 nm process are _not_ 3 nm across. That said, they are still really small.

[^4]: I do mean high-level - these are the components of the pipeline, but not arranged how they are actually laid out on the chip. Also, the P5 microarchitecture has an on-chip floating point unit which is not shown here. Due to how x87 floating point works, that unit has its own registers and pipeline. There's also a lot of additional complexity for virtual addressing and cache management, which we aren't worried about here.

[^5]: "Scalar" in this context means "one instruction per cycle."

[^6]: "Logically" means that it came second in the input program. This is in constrast to _architecturally_, where it is simultaneous with the first `mov` instead of after it.

[^7]: Actually (because of course there's an "actually"), x86's `ret` instruction can optionally take an integer operand. When it does, instead of only popping a return address off the stack, it will pop the given amount of extra space _before_ popping the return address. This is not common and would make `sp` prediction harder. The P5 doesn't do it. Such `ret` instructions remain unpairable if they are involved in hazards on `sp`.