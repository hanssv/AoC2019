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

```
218> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/01/sol_01.aes", "solve_1", []).
4034 steps / 43375 gas / 1142144 reductions / 26.50ms
3305041
219> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/01/sol_01.aes", "solve_2", []).
13993 steps / 162237 gas / 4047248 reductions / 85.79ms
4954710
```

### [Day 2](02/sol_02.aes)

The [second problem](https://adventofcode.com/2019/day/2) introduce the
`Intcode` virtual machine, where a sequence of integers should be interpreted
as op-codes and arguments. The first version of this machine only does `add`
and `mul` but I have a feeling this machine will come back in later problems.
The first half of the puzzle was trivial, the second half took ages to compute
(not to implement). But after I had up:ed the gas limit (see below) and split
the setup of the `Intcode` machine memory into setup and initialization it can
be run in about 15 seconds on a normal laptop. (Consuming a measly 835655389
of gas...)

```
226> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/02/sol_02.aes", "solve_1", []).
1762 steps / 19026 gas / 435334 reductions / 9.33ms
8017076
227> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/02/sol_02.aes", "solve_2", []).
2071337 steps / 835655389 gas / 612813613 reductions / 13215.18ms
3146
```

### [Day 3](03/sol_03.aes)

The [third problem](https://adventofcode.com/2019/day/3) was a grid puzzle. Two
long and twisting wires should be analyzed for crossings. And the task was to
return the crossing nearest to the starting point (and the crossing with
shortest steps from the starting point in part 2). For efficiency reasons I
decided to sacrifice compact code for repeating myself a bit. There is an
overlap between part 1 and part 2, but the solution is basically repeated
twice. In the end it solves the problem for my input in about 20 seconds.

```
223> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/03/sol_03.aes", "solve_1", []).
3409823 steps / 3246947600 gas / 872551327 reductions / 19177.61ms
{tuple,{245,0}}
224> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/03/sol_03.aes", "solve_2", []).
4652200 steps / 8351345675 gas / 1184794858 reductions / 26065.90ms
{tuple,{48262,{tuple,{1002,68}}}}
```

### [Day 4](04/sol_04.aes)

The [fourth problem](https://adventofcode.com/2019/day/4) was about checking
integer "passwords" for some somewhat strange properties. The straightforward
solution looping over all integers and checking them one by one worked. But it
was really slow (upwards of two minutes) - so in the end I decided to use one
of the properties (digits must be increasing) to skip over many integers at the
same time. This improved the solution a lot, and it now even fits in a single
block (5.7M gas) and runs in about half a second.

```
232> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/04/sol_04.aes", "solve_1", []).
154101 steps / 5713303 gas / 43829631 reductions / 879.54ms
979
233> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/04/sol_04.aes", "solve_2", []).
160913 steps / 5895972 gas / 45769517 reductions / 928.78ms
635
```

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

```
242> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/05/sol_05.aes", "solve_1", []).
8239 steps / 97363 gas / 2078951 reductions / 42.48ms
[0,0,0,0,0,0,0,0,0,7988899]
243> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/05/sol_05.aes", "solve_2", []).
9792 steps / 117144 gas / 2519916 reductions / 49.11ms
[13758663]
```

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

```
244> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/06/sol_06.aes", "solve_1", []).
122224 steps / 5762478 gas / 36147472 reductions / 799.13ms
261306
245> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/06/sol_06.aes", "solve_2", []).
60735 steps / 1607409 gas / 17312423 reductions / 390.45ms
382
```

### [Day 7](07/sol_07.aes)

The [seventh problem](https://adventofcode.com/2019/day/7) was yet another
twist of the `Intcode` machine. This time we needed to handle multiple
"processes" where output from one process was wired to input of another
process. To achieve this I had to do a bit of refactoring since now execution
of a program must be able to stop and wait for more input. The problem was more
tedious than challenging - and in the end part 2 could be run in around 15
seconds.

```
246> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/07/sol_07.aes", "solve_1", []).
411264 steps / 47687698 gas / 117140735 reductions / 2413.68ms
Store: #{1 => #{}}
247> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/07/sol_07.aes", "solve_2", []).
2377824 steps / 1366772167 gas / 668947268 reductions / 14100.08ms
{tuple,{2299406,[6,5,9,7,8]}}
```

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
