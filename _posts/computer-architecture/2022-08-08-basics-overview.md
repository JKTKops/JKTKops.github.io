---
layout: post
title: "Crash Course on Principles of Computer Architecture"
date: 2022-08-08 21:00:00 -5
categories: computer-science hardware
---

In this category of posts, we're going to talk about computer architecture.

For completeness, before getting into any of the crazy (and crazy _cool_!) modern technologies,
I want to give a crash-course overview of the basics of computer architecture. How do computers
work at the logical level?

Most posts in this series are going to be deep dives into a particular technology, how it works,
and how it might be implemented. However, I don't feel justified in getting into that without
having an accompanying crash course on the pre-requisites.

This post is going to zoom through what should be a one-semester undergraduate university course.
It won't be comprehensive, but should get the ideas across.

* TOC
{: toc}

# What is a Computer?

It's easy to see computers as magical black boxes. One common joke is that "Computers are just
rocks we tricked into thinking." And that's actually sort of true. But computers don't _think_
the way we do. Really, computers are nothing more than glorified calculators. The images you
see on your screen are the results of (sometimes complex) calculations to produce location
and color data for each pixel. Everything comes down to moving around data and performing
arithmetic on that data.

A typical definition of a computer is "a machine that can be programmed to carry out sequences
of arithmetic or logical operations automatically." We might ask what can be computed in this
way; namely: if I give you a function definition, and some inputs to that function, what types
of computers are capable of evaluating the function on those inputs? The study of different
types of computers is called _models of computation_. The highest class of model of computation
contains "computers that can compute anything which _can_ be computed."[^1] Those are what we're
concerned with here.

# Capabilities

A computer has to be programmable by the definition above. Computers will get their programs as
a sequence of steps to perform. Each step tells the computer to use a single one of its capabilities.
Computers have some short-term storage called _registers_. They can operate directly only on values
that are in registers.

The most basic capabilities of a modern computer all fall into one of the following categories:

1. Retrieve data from memory into a register ("memory load")
2. Place data into memory from a register ("memory store")
3. Perform arithmetic (or logic) on two numbers in registers
4. Redirect to a different location in the sequence of steps, for example "Go to step 4."
5. _Conditionally_ redirect, for example "Go to step 4, but only if x is 0."

After performing one step, we move on to the next step in the sequence.

Any particular computer has a particular set of instructions that it can understand, called
an "instruction set," or ISA[^2]. Of course,
it doesn't understand them written in English. They have to be encoded in binary in a consistent
way which depends on the particular computer. We're not going to concern ourselves with that
encoding here. We'll assume the existence of a circuit called a "decoder," which translates the
binary-encoded instruction into a bunch of "control signals" which control what the rest of the
computer does.

We're also not going to worry about modelling a computer that can compute _quickly_. Computing
at all will do for now, and we'll worry about being fast later.

Common instructions available in most instruction sets are:

1. LDI (or "Load Immediate"): place a specified value into a specified register[^3]
2. ADD: add the values in two specified registers, storing the result into a third specified register
3. Other arithmetic or logic operations, for example AND, OR, and SUB
4. JMP: redirect to a given instruction in the program[^4]
5. BRANCH: conditionally redirect, using a specified condition to test a specified value.

So what do we need to implement all of this?

1. Somewhere to store the program
2. Some collection of registers (typically called a "Register File")
3. Some type of data memory
4. Circuitry that can perform arithmetic and logic (typically called an Arithmetic/Logic Unit,
    or ALU)
5. A counter to keep track of where we are in the program
6. A way to overwrite the counter
7. Circuitry that can test conditions

# A Simple Model

Here's a very basic way we could combine the above pieces into something resembling a computer.

<center>
<img src="/assets/architecture/basic-computer.svg">
</center>

The decoder controls what everything else does. It tells memory to load or store (or do nothing),
it tells the register file which registers to read and write, the ALU what operation to perform,
the test unit what logical test to use, and the program counter whether or not it should overwrite
and with what value (possibly depending on the result of the test, which comes from the test unit).

This is actually a perfectly good basic model of a computer. Modern computers don't look like this,
but they have components that are individually recognizable as components of this model.

## Performance

A computer based on this model is going to be bad. Why?

Each "cycle" of the computer is controlled by a clock. When the clock ticks, we move on to the next
step according to the program counter. We need to give each step enough time for all of the
control and data signals to propogate through the whole system. For some of these signals, that
might be slow[^5]. Say the slowest signal chain starts from the program counter, goes through the
program, decoder, register file, ALU, and back to the register file (perhaps for an operation
like division, which is fairly slow to perform). This slowest chain is called the _critical path_.
If it takes half a second for a signal to get all the way through the critical path, then we cannot
run our clock any faster than 0.5s/cycle (alternatively, 2 cycles per second or 2 Hz).

Even though some, or even most, instructions won't use the critical path, any instruction _might_
be the instruction that does. So the critical path is the limiting factor to our clock speed.
The clock speed is a significant factor when considering the speed of a computer. A faster clock
will almost always mean a faster computer[^6].

So the first way we can think of to improve the clock speed is to make the critical path shorter.

# Pipelining

The major improvement to the basic design, used in all modern systems, is called _pipelining_.
We don't want to limit our system to only working on a single instruction at a time, and being
stuck until that instruction is done. We also want to make the critical path shorter. We can
hit both birds with one stone: split up the execution of an instruction into several "stages,"
where each stage takes one clock cycle.

Since an instruction can only occupy one stage at a time, if we have 3 stages then we can
also be "executing" 3 instructions at once. We also run the clock faster[^7], but this is
negated by the fact that it also takes more cycles for an individual instruction to finish.

These are the relevant numbers concerning a pipeline:

1. Clock speed: how many clock cycles we get per second
2. Latency: how many clock cycles it takes to execute one instruction from start to finish
3. Throughput: how many instructions complete each clock cycle

For our basic model here, the throughput should stay at 1, even though the latency has increased.
Since we're running the clock faster, that means we're finishing more instructions _per second_,
so our computer is going faster. In an ideal world, the latency triples, the clock speed gets cut
into 1/3, and the throughput remains 1. That works out to our computer running about three times
faster, without any changes to the program!

To implement pipelining, we use "pipeline registers," also called "fences," to separate each
stage. We can modify the basic design to something like this, for a 3-stage example.

<img src="/assets/architecture/basic-pipeline.svg">

This gives us a _mostly_ clean division between each stage of the pipeline - fetch, execute,
and writeback. But hang on - stage 3 doesn't appear to exist at all!

Stage 3 is the "writeback" stage, where the results are written to where they have to go, which
is either the register file or program counter (memory writes are handled in the second stage,
which is called "execute"). What logic there is to handle in this stage would be built into
the register file and program counter, so there's no new units here. But that does mean
there isn't a clear separation between stages. As you might expect, that causes major problems.

Consider the instruction sequence that adds register A to register B, storing to register C,
and then adds register C to register D, storing back to register A. We can visualize what's
going on with a _timing table_ as follows:

$$\begin{array}{|c|c|c|c|}
\hline
\text{stage} & \text{cycle 1} & \text{cycle 2} & \text{cycle 3} & \text{cycle 4} \\
\hline
\text{Fetch} & \mathtt{ADD\ A\ B\ C} & \tt{ADD\ C\ D\ A} & - & - \\
\hline
\text{Execute} & - & \tt{ADD\ A\ B\ C} & \tt{ADD\ C\ D\ A} & - \\
\hline
\text{Write} & - & - & \tt{ADD\ A\ B\ C} & \tt{ADD\ C\ D\ A} \\
\hline
\end{array}$$

This table shows us the instruction in each stage at any given cycle. We could also write
these as instructions vs time, instead of stage vs time, but I find this way a bit easier.

There's a big problem with this program. The result of the first instruction won't be available
in the register file until _after_ the cycle that the instruction writes back. But looking at
cycle 3 in the table, the second instruction needs to read register C as input on the same cycle!

So what gives, and how can we fix it?

## Data Hazards

The above problem is known as a _data hazard_, specifically of the read-after-write kind. This
is often abbreviated RAW. Hazards describe types of _data dependencies_, where the order of
instructions in the program implicitly connect instructions which write and read the same data
from a register. A RAW hazard means that we must not read the register until after the write
has completed.

There are other types of hazards as well. If we have a write-after-read hazard, then we must
ensure that the instruction which reads is able to read the data _before_ it is overwritten
by the write. There are also write-after-write hazards, which require us to enforce that
results are written in the correct order so that the correct data is in registers after
the instruction sequence.

Since our model executes instructions in order[^8], WAR and WAW hazards simply cannot happen.

## Resolving RAW Hazards

There are two ways we could try to resolve the RAW hazard in the above program.

The first, and most obvious solution, is to force `ADD C D A` to wait until `ADD A B C` completes
writeback before allowing it to execute. This is tricky to implement, however, because up to
this point we have always assumed that instructions move through the pipeline at exactly one
stage per cycle - no more, no less. However it's certainly possible and this approach is known as
a "pipeline stall."

The better solution is to provide a _shortcut_ for `ADD C D A`'s writeback so that it is accessible
to an executing instruction in the same cycle. This technique is called _data forwarding_.

Either way, we have to detect that two instructions have a hazard; this is generally easy as we
can just remember which register(s) are used by every instruction and compare the registers being
read in the execute stage to the registers being written in the write-back stage.

## Method 1: Pipeline Stall

Using the first approach, we need to modify our first pipeline fence so that it gets an extra
control signal. This signal describes if it should output the stored instruction at all. If it
shouldn't, it should instead output a `NOP` instruction, which is short for "no operation."

`NOP`s which enter the pipeline due to stalls are commonly called _pipeline bubbles_, since they
behave identically to air bubbles in a water pipe.

Additionally, if we stall, we need to prevent the program counter from advancing, or else we will
lose the stalled instruction.

Adding a hazard control unit to our model, we can come up with a computer design like the following.

<img src="/assets/architecture/stall-pipeline.svg">

With this approach, the same program experiences the following timing table:

$$\begin{array}{|c|c|c|c|}
\hline
\text{stage} & \text{cycle 1} & \text{cycle 2} & \text{cycle 3} & \text{cycle 4} & \text{cycle 5}\\
\hline
\text{Fetch} & \mathtt{ADD\ A\ B\ C} & \tt{ADD\ C\ D\ A} & \tt{ADD\ C\ D\ A} & - & - \\
\hline
\text{Execute} & - & \tt{ADD\ A\ B\ C} & \tt{NOP} & \tt{ADD\ C\ D\ A} & - \\
\hline
\text{Write} & - & - & \tt{ADD\ A\ B\ C} & \tt{NOP} & \tt{ADD\ C\ D\ A} \\
\hline
\end{array}$$

I think once these tables get complicated, they are easier to read if we put the pipeline on
the horizontal axis, to match the pipeline diagrams. Let's do that from now on; here's the
same table transposed.

$$\begin{array}{|c|c|c|c|}
\hline
\text{cycle} & \text{Fetch} & \text{Execute} & \text{Write} \\
\hline
1 & \tt{ADD\ A\ B\ C} & - & - \\
\hline
2 & \tt{ADD\ C\ D\ A} & \tt{ADD\ A\ B\ C} & - \\
\hline
3 & \tt{ADD\ C\ D\ A} & \tt{NOP} & \tt{ADD\ A\ B\ C} \\
\hline
4 & - & \tt{ADD\ C\ D\ A} & \tt{NOP} \\
\hline
5 & - & - & \tt{ADD\ C\ D\ A} \\
\hline
\end{array}$$

We can see that the `NOP` bubble causes the whole program to take an extra cycle, as expected.

## Method 2: Data Forwarding

Instead, let's use a _forwarding unit_ behind the register file to detect these hazards, and
forward data from the writeback stage. We get a design that looks as follows.

<img src="/assets/architecture/forward-pipeline.svg">

Now we get a timing table which is the same as the original one, except this time it works!

There is a potential complication with this design though. Often, the critical path of the
system with this type of design is _already_ in the execute stage (although this is by no means
a guarantee). The Forwarding Unit, while cheap, does introduce some extra signal delay and
can make the critical path longer, slowing down the clock.

Additionally, this plan only works if the writeback stage is immediately after the stage
performing the read. If retrieving instructions from the program is fast, we may opt to place the
fetch-execute fence so that the register file read happens in the fetch stage instead. This
would mean there is a two cycle gap between register reads and register writes, so two adjacent
instructions with a RAW dependency cannot use this forwarding scheme directly. Multi-stage
separation like this is pretty much always the case in real designs. Ideally, we combine both
approaches. We stall exactly long enough for the data to be available for forwarding.

# Pipelining With Control Flow

That simple example program didn't make any use of the control flow capabilities of the computer -
(conditionally) jumping. Let's look at what happens if we execute a simple program like this.

```mips
ADD A, B, C
JMP 2       # jump forward two instructions
ADD B, C, D # this should be skipped
SUB C, B, A
```

Remember that the control signals for jump instructions go through the Test unit, which is
in the execute stage. Perhaps you can already see where this is going! Here's the timing table.

$$\begin{array}{|c|c|c|c|}
\hline
\text{cycle} & \text{Fetch} & \text{Execute} & \text{Write} \\
\hline
1 & \tt{ADD\ A,\ B,\ C} & - & - \\
\hline
2 & \tt{JMP\ 2} & \tt{ADD\ A,\ B,\ C} & - \\
\hline
3 & \tt{ADD\ B,\ C,\ D} & \tt{JMP\ 2} & \tt{ADD\ A,\ B,\ C} \\
\hline
4 & \tt{SUB\ C,\ B,\ A} & \tt{ADD\ B,\ C,\ D} & \tt{JMP\ 2} \\
\hline
5 & \tt{SUB\ C,\ B,\ A} & \tt{SUB\ C,\ B,\ A} & \tt{ADD\ B,\ C,\ D} \\
\hline
6 & - & \tt{SUB\ C,\ B,\ A} & \tt{SUB\ C,\ B,\ A} \\
\hline
7 & - & - & \tt{SUB\ C,\ B,\ A} \\
\hline
\end{array}$$

Oh no! The `ADD B, C, D` instruction and even an extra copy of the `SUB` instruction,
snuck into the pipeline before we realized we were supposed to skip forward!

What gives, and how do we fix it?

## Control Hazards

The fact that there is time between fetching a jump (or branch) instruction, and actually
redirecting the program counter, means that there will always be a chance for instructions that
should have been skipped to sneak into the pipeline. This is called a _control hazard_.

Since we're not omniscient, and neither is our computer, there's not really anything we can
do to make the program go faster in every case, unlike with the RAW hazards. In future posts,
we'll see some methods that can work in _most_ cases.

The simplest approach is after decoding a jump (or branch) instruction, we simply stall until
it completes execution and then continue after being redirected. This, obviously, introduces
large pipeline bubbles and is generally not ideal.

An easy potential improvement to that is to have separate data paths (signal paths through
the circuit) for jump and branch instructions, since jumps can be detected and executed
much more easily. In our simple computer architecture, we can most likely detect jump instructions
directly in the decoder and execute them in the Fetch stage without causing critical path issues.

A harder improvement is to consider that, sometimes, the instruction that would sneak into the
pipeline actually _should_ be the next instruction. We don't know it yet, but we could hope to
get lucky. When the branch actually executes, if the branch is in fact taken, we have to track
down those "hopeful" instructions and remove them from the pipeline. For our simple pipelines,
this is easy - it's every instruction in the pipeline in an earlier stage than the branch.
We remove those instructions by replacing them with `NOP`s, which is called _squashing the pipeline_.

We can implement that in our pipeline registers by adding some control ability. When we need to
stall, the fence currently (1) does not read new input from the previous stage, and (2) does
not send its stored instruction to the next stage (sending a `NOP` instead). In order to squash,
the fence should replace the stored instruction with a `NOP` instead of reading new input from
the previous stage. Next cycle, after the program counter redirect, the Fetch stage will contain
the correct next instruction, prepared to send it into the first fence. The squashed instructions
have all become `NOP`s.

An architecture block diagram and timing table for the above program with this scheme could look
like this.

<img src="/assets/architecture/speculative-pipeline.svg">

$$\begin{array}{|c|c|c|c|}
\hline
\text{cycle} & \text{Fetch} & \text{Execute} & \text{Write} \\
\hline
1 & \tt{ADD\ A,\ B,\ C} & - & - \\
\hline
2 & \tt{JMP\ 2} & \tt{ADD\ A,\ B,\ C} & - \\
\hline
3 & \tt{SUB\ C,\ B,\ A} & \tt{NOP} & \tt{ADD\ A,\ B,\ C} \\
\hline
4 & - & \tt{SUB\ C,\ B,\ A} & \tt{NOP} \\
\hline
5 & - & - & \tt{SUB\ C,\ B,\ A} \\
\hline
\end{array}$$

We executed the unconditional jump immediately in the fetch stage (and have the decoder
pass on a `NOP` instead of the now-useless `JMP`). As a result, the erroneous instruction
never enters the pipeline at all, and we even finish two cycles faster. Nice!

What if that unconditional jump was a conditional branch? Let's not worry about types of
conditional branches here; let's just assume that the same `JMP 2` instruction now needs
to use the Test unit. We get a timing table like this one.

$$\begin{array}{|c|c|c|c|}
\hline
\text{cycle} & \text{Fetch} & \text{Execute} & \text{Write} \\
\hline
1 & \tt{ADD\ A,\ B,\ C} & - & - \\
\hline
2 & \tt{JMP\ 2} & \tt{ADD\ A,\ B,\ C} & - \\
\hline
3 & \tt{ADD\ B,\ C,\ D} & \tt{JMP\ 2} & \tt{ADD\ A,\ B,\ C} \\
\hline
4 & \tt{SUB\ C,\ B,\ A} & \tt{ADD\ B,\ C,\ D} & \tt{JMP\ 2} \\
\hline
5 & \tt{SUB\ C,\ B,\ A} & \tt{NOP} & \tt{NOP} \\
\hline
6 & - & \tt{SUB\ C,\ B,\ A} & \tt{NOP} \\
\hline
7 & - & - & \tt{SUB\ C,\ B,\ A} \\
\hline
\end{array}$$

As before, we still let the erroneous `ADD` and `SUB` instructions into the pipeline,
but now we squash them when the `JMP` instruction goes to write back. Since they never
reach the writeback stage themselves, the values they computed are never stored in the
register file. It's like the instructions never existed at all.

A more advanced technique called _branch prediction_ attempts to guess whether or not
a branch will be taken as soon as it is decoded, and gets the next instruction from the
predicted location in the program. What we're doing is the same as predicted that all branches
are not taken.

It turns out that accurate branch prediction is _extremely_ important for performance in even
a moderately deep pipeline, because _mispredicting_ a branch introduces a bubble in the pipeline
of length equal to the number of stages between the program counter and whichever stage is able
to detect the misprediction (in our case, this is writeback, but with some effort, complex designs
can do it in the execute stage). A modern x86 processor has a pipeline with around 12 stages in
the relevant portion of the pipeline, so the misprediction penalty is _huge_.

Branch prediction is a difficult problem, with an incredibly rich field of results and techniques.
Modern branch predictors are _incredibly_ accurate, achieving prediction accuracies in the
neighborhood of 99%. Methods of branch prediction will be the topic of several future posts!

# Memory is Slow

There's a common joke around the internet about how Internet Explorer takes 10x as long as every
other browser to serve a webpage. Memory is like the Internet Explorer of execution units. It takes
_much_ longer to read or write to RAM than to execute any other type of instruction.[^9] Given
that we need to access RAM whenever we finish each small unit of computation, it's completely
unacceptable to spend over 99% of program execution time sitting around waiting for RAM.

The idea of having registers as short-term, fast storage is so good that we can abuse it to
solve this problem too.

Instead of talking directly to RAM whenever we need to access it, we use a middle-man memory
unit called a _cache_. Just like caching in your web browser, the cache in a CPU remembers data
that it recently had to get from memory. Since the cache is smaller and closer to the CPU than
RAM, it is much faster to access. If the memory we're looking for is already in the cache, we
can retrieve it in usually just a single cycle. This is called a "cache hit." In the event of a
cache miss, we only have to pay the huge penalty for accessing RAM one time, and then future
references to the same data will go through the cache.

When a program access some memory, it will almost always access the same memory or nearby memory
soon after. Programs accessing nearby memory is called "spatial locality," while the fact that
those accesses are typically soon after each other is called "temporal locality." Programs
written to maximize spatial and temporal locality of memory accesses are likely to perform
better if the computer has a cache.

Cache design is a whole can of worms; figuring out the interaction size between cache and RAM
(how much data to retrieve _surrounding_ the requested data, assuming it will be needed soon),
as well as the size of the cache itself, and when to evict cache entries to make room for new ones,
are all important considerations. Additionally, most designs will use multiple layers of cache,
keeping a smaller, extremely fast cache directly next to the execution circuitry, and a larger
but somewhat slower cache near the edge of the CPU core. Multi-core systems often have a _third_
layer, with a significantly larger and slower cache shared by all or several cores.

We're not going to get into it here, but it is important to be aware that caches exist and that
they are not a one-size-fits-all solution to slow memory problems. On real hardware, writing
programs in a "cache-aware" fashion, but without changing the underlying algorithm, can create
performance improvements large enough to be noticeable by a human.

# Conclusions

From the outside, a computer is simply a black box which takes in an algorithm, executes it
step-by-step, and spits out a result. In actuality, there are a huge host of techniques
we can use to maintain this outside appearance, but achieve the same goals much, much faster.

Pipelining lets us re-use existing hardware on several instructions at the same time, slowing
down individual instructions but speeding up the clock and therefore also how many instructions
complete per second. It introduces challenges in maintaining the data-flow and control-flow
of the original program, but these challenges can be overcome without too much difficulty.

We've also seen how we can use even basic branch prediction techniques to eliminate branch
stalls if we're able to "get lucky," and hinted at the possibility that if we try very, very
hard, we can "get lucky" almost every time.

Finally, we've briefly discussed how slow memory is, and how we can use _caching_ techniques
to eliminate the large stalls associated with waiting for memory accesses.

This crash-course overview covers more or less the same topics as a one-semester undergraduate
computer architecture course. To avoid getting too mathematical, I covered branches in significantly
less detail than such a course would.

In future posts, we will take deep dives into particular advanced computer architecture techniques.
We'll look at other pipeline designs, particularly ones that can execute instructions out of
order. We'll also see various techniques for branch prediction, and discuss the design and
construction of caches in significantly more detail.

See you next time!

---------------

[^1]: It is surprisingly easy to prove that not everything can be computed, by constructing an
      explicit example of a function which cannot be computed by any algorithm. The YouTube
      channel "udiprod" has a [nice approachable video on this](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjhgrS_grj5AhXPAjQIHTIFBEYQwqsBegQINxAB&url=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3D92WHN-pAFCs&usg=AOvVaw2ovHeA16DSOFBsUvypP_rT)
      which I can highly recommend.

[^2]: The A stands for Architecture, because an ISA is generally considered half the design of
      a computer. The other half is the hardware that interprets the instructions and does
      what they say.

[^3]: Depending on the ISA, the register may be specified by the ISA itself (for example,
      defining LDI to always place the value into register 0) or it may be specified as part
      of the particular instruction. The same goes for all references to "specified register"
      here.

[^4]: Where to jump to can be specified as an "absolute," for example "go to step 4," or as
      a _relative_, for example "go back 3 steps." Which one is used depends on the particular
      ISA, and many ISAs use both for different instructions.

[^5]: In particular, accessing memory is _very_ slow, but we're not going to worry about
      that yet.

[^6]: Up to a point. At some point, the fact that memory is so slow becomes the limiting
      factor to how fast we can operate on data. The computer would be able to work on
      any data as it comes from memory and be done before the next data is ready from
      memory. Empirically, this happens in the neighborhood of 4 billion cycles per
      second, or 4 GHz. Modern computers have clocks running at about that speed.

[^7]: Up to 3 times faster, though in practice the critical paths of the individual stages
      will usually not each be exactly one third of the critical path of the non-pipelined
      design.

[^8]: The implication being that there are models which execute instructions _out of order_,
      and that is precisely why I'm writing this series.

[^9]: In the neighborhood of _200_ clock cycles.
