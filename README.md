# Advent of Code - 2019

This repository contains solutions to the [Advent of Code
2019](https://adventofcode.com/2019). There are many such repositories floating
around, the twist in this case is that the solutions are written in
[Sophia](https://github.com/aeternity/protocol/blob/master/contracts/sophia.md)
the smart contract language for the [Aeternity
blockchain](https://aeternity.com/).

### [Day 1](01/sol_01.aes)

This year's Advent of Code [started](https://adventofcode.com/2019/day/1) with
a straightforward problem. You had to create fuel consumption for a number of
objects depending on their mass. The second part of the problem also included
gas for the mass of the gas required, making the problem (trivially) recursive.

There were no problems writing a [solution](01/sol_01.aes) in Sophia, the
recursion is done with an accumulator to save some gas.

### [Day 2](02/sol_02.aes)

The [second problem](https://adventofcode.com/2019/day/2) introduce the
`Intcode` virtual machine, where a sequence of integers should be interpreted
as op-codes and arguments. The first version of this machine only does `add`
and `mul` but I have a feeling this machine will come back in later problems.
The first half of the puzzle was trivial, the second half took ages to compute
(not to implement). But after I had up:ed the gas limit (see below) and split
the setup of the `Intcode` machine memory into setup and initialization it can
be run in about 20 seconds on a normal laptop. (Consuming a measly 1503823113
of gas...)

### [Day 3](03/sol_03.aes)

The [third problem](https://adventofcode.com/2019/day/3) was a grid puzzle. Two
long and twisting wires should be analyzed for crossings. And the task was to
return the crossing nearest to the starting point (and the crossing with
shortest steps from the starting point in part 2). For efficiency reasons I
decided to sacrifice compact code for repeating myself a bit. There is an
overlap between part 1 and part 2, but the solution is basically repeated
twice. In the end it solves the problem for my input in about 20 seconds.

### [Day 4](04/sol_04.aes)

The [fourth problem](https://adventofcode.com/2019/day/4) was about checking
integer "passwords" for some somewhat strange properties. The straightforward
solution looping over all integers and checking them one by one worked. But it
was really slow (upwards of two minutes) - so in the end I decided to use one
of the properties (digits must be increasing) to skip over many integers at the
same time. This improved the solution a lot, and it now even fits in a single
block (5.7M gas) and runs in about half a second.

### [Day 5](05/sol_05.aes)

The [fifth problem](https://adventofcode.com/2019/day/5) marked the return of
`Intcode` machine. Now it should be extended with immediate arguments, as well
as input, output, jumps and conditionals. Not being very computationally
expensive it was entirely straightforward to extend the solution from [Day
2](#day-2) by adding plumbing for inputs and outputs and adding the new
opcodes.

At the end I experimented a bit with various optimizations, and I noticed that
more aggressive inlining sometimes made a difference. For example inlining all
the calls to `val/3` saves 6.5% of the gas. Still today's problems would fit
comfortably within a microblock with 97k/117k gas for the part 1/part 2.

### [Day 6](06/sol_06.aes)

The [sixth problem](https://adventofcode.com/2019/day/6) included a fairly
simple graph/tree problem. In the first part you were supposed to find the
total depth of all nodes in the tree and in the second part you had to find the
common ancestor of two nodes and calculate the distance between them (through
that common ancestor).

This problem showed a shortcoming of Sophia, we are missing some functions on
`string`, to handle the raw input data we would have needed `String.split`
which is [not yet there](https://github.com/aeternity/aesophia/issues/176). So
we did some preprocessing of the input data (you could do this and write it to
the state, so not to much cheating I think).

The naive implementation of part 1 became pretty expensive 3.5G gas and more
than 15 seconds runtime, but after adding some memoization (using the state!)
the solution actually fits in a block at 5.7M gas. Part 2 was a lot easier and
only require 1.6M gas.

### [Day 7](07/sol_07.aes)

The [seventh problem](https://adventofcode.com/2019/day/7) was yet another
twist of the `Intcode` machine. This time we needed to handle multiple
"processes" where output from one process was wired to input of another
process. To achieve this I had to do a bit of refactoring since now execution
of a program must be able to stop and wait for more input. The problem was more
tedious than challenging - and in the end part 2 could be run in around 15
seconds.


## Running contracts
For obvious reasons I don't want to run these contracts on-chain, and I don't
even want to run them via contract calls etc. Instead we re-use part of the
test framework in this way:

```
~/Quviq/Aeternity: git clone git@github.com:aeternity/aeternity.git
...

~/Quviq/Aeternity: cd aeternity
~/Quviq/Aeternity/aeternity: ./rebar3 as test shell --apps=""
===> Verifying dependencies...
...

(aeternity_ct@localhost)1> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/01/sol_01.aes", "solve_1", []).
4034 steps / 43375 gas / 163948 reductions / 2.47ms
Store:
  #{}
3305041
(aeternity_ct@localhost)2>
```

Also for some of the later problems I had to change the value of the `CALL_GAS` macro
in `apps/aefate/test/aefa_sophia_test.erl` to allow my contract to use more gas :-)

```
~/Quviq/Aeternity/aeternity: git diff apps/aefate/test/aefa_sophia_test.erl
diff --git a/apps/aefate/test/aefa_sophia_test.erl b/apps/aefate/test/aefa_sophia_test.erl
index 75135681..7e2a36df 100644
--- a/apps/aefate/test/aefa_sophia_test.erl
+++ b/apps/aefate/test/aefa_sophia_test.erl
@@ -106,7 +106,7 @@ compile_contract(Code, Options) ->
         {error, {type_errors, Err}}
     end.

--define(CALL_GAS, 6000000).
+-define(CALL_GAS, 6000000 * 10000).

 make_store(<<"init">>, _) -> aefa_stores:initial_contract_store();
 make_store(_, none) ->
```
