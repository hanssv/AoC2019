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

### [Day 8](08/sol_08.aes)

The [eighth problem](https://adventofcode.com/2019/day/8) came down to turning
a very large number (or a stream of digits, depending on your interpretation)
into image layers. Part 1 was to produce a strange checksum and part 2 was to
flatten the image layers, getting rid of transparent pixels.

Normally you'd processed the input by turning it into a list of characters, but
that doesn't really fly in Sophia. So instead I made use of the fact that we
have unbound integers and represented the input as a huge number and produces
the layers and individual pixels by `div` and `mod`. Not the most efficient,
but a straightforward solution that ran in 2 and 6 seconds respectively for
part 1 and 2. I was bit short on time, so this solution isn't very optimized,
nor very elegant.

```
296> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/08/sol_08.aes", "solve_1", []).
314569 steps / 44012413 gas / 78303937 reductions / 1567.86ms
{tuple,{8,1792}}
297> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/08/sol_08.aes", "solve_2", []).
1142752 steps / 279521657 gas / 316914695 reductions / 6667.20ms
[[1,0,0,0,0,0,0,1,1,0,1,1,1,1,0,0,1,1,0,0,1,0,0,1,0],
 [1,0,0,0,0,0,0,0,1,0,1,0,0,0,0,1,0,0,1,0,1,0,0,1,0],
 [1,0,0,0,0,0,0,0,1,0,1,1,1,0,0,1,0,0,0,0,1,1,1,1,0],
 [1,0,0,0,0,0,0,0,1,0,1,0,0,0,0,1,0,0,0,0,1,0,0,1,0],
 [1,0,0,0,0,1,0,0,1,0,1,0,0,0,0,1,0,0,1,0,1,0,0,1,0],
 [1,1,1,1,0,0,1,1,0,0,1,1,1,1,0,0,1,1,0,0,1,0,0,1,0]]
```

### [Day 9](09/sol_09.aes)

The [ninth problem](https://adventofcode.com/2019/day/9) was yet another twist
to the `Intcode` machine. This time relative addressing was introduced, and I
decided (since we didn't need more than one process) to build it on top of the
solution from [Day 5](05/sol_05.aes). Implementation was entirely
straightforward. Two complications that were mentioned in the problem
descriptions, large numbers and memory addressing, was entirely trivial since
we have large numbers built-in and were using a map for memory.

Part 2 took a long time to run, around 2 minutes, but with inlining and some
other tweaks (not part of the presented solution) it could be brought down to
about one minute...

```
352> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/09/sol_09.aes", "solve_1", []).
19006 steps / 249757 gas / 5034295 reductions / 101.50ms
[2955820355]
353> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/09/sol_09.aes", "solve_2", []).
18660835 steps / 31690694258 gas / 5424189214 reductions / 121683.58ms
[46643]
```

### [Day 10](10/sol_10.aes)

The [tenth problem]((https://adventofcode.com/2019/day/10) was about line of
sight in an asteroid field. In part 1 we were tasked to find the asteroid that
had most asteroids in line of sight. Again we were handicapped by not having
any string processing functions so I transformed the input into a list of list
of 0's and 1's.

The algorithm is nothing fancy, for each asteroid find the line of sight vector
to each other asteroid (normalized) and then count the number of unique
vectors. The problem had ~400 asteroids and this ran to completion in about 15
seconds. In part 2 we were using a laser to blast the asteroids (laser rotating
clockwise) and the task was to find which asteroid would be the 200th to be
evaporated. The implementation actually computes the entire list in which order
the asteroids are targeted, but since 200 is less than the answer to part 1 we
could have gotten away with a less complex solution. This is quick to compute,
less than half a second.

```
99> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/10/sol_10.aes", "solve_1", []).
10125376 steps / 8232360098 gas / 425113858 reductions / 14420.50ms
{tuple,{334,{tuple,{23,20}}}}
100> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/10/sol_10.aes", "solve_2", []).
309995 steps / 10060411 gas / 12705904 reductions / 371.74ms
1119
```

### [Day 11](11/sol_11.aes)

The [eleventh problem](https://adventofcode.com/2019/day/11) was yet another
usage of the `Intcode` machine. This time though the problem was merely using
it, so no changes necessary to that part. The task was to build a painting
robot, that moved along an infinite grid, running an `Intcode` program for each
square deciding on the color of the square and the direction to turn in. For
part 1 you only had to count the number of squares that was painted but for
part 2 we needed to display what was painted so some rudimentary string
processing was done in Sophia!

```
220> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/11/sol_11.aes", "solve_1", []).
5568706 steps / 4264437725 gas / 237784397 reductions / 6976.82ms
2319
221> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/11/sol_11.aes", "solve_2", []).
747875 steps / 62233696 gas / 30851420 reductions / 636.95ms
[<<" #  # #### ###  ###  ###  ####  ##    ##   ">>,
 <<" #  # #    #  # #  # #  # #    #  #    #   ">>,
 <<" #  # ###  #  # #  # #  # ###  #       #   ">>,
 <<" #  # #    ###  ###  ###  #    # ##    #   ">>,
 <<" #  # #    # #  #    # #  #    #  # #  #   ">>,
 <<"  ##  #### #  # #    #  # #     ###  ##    ">>]
```

### [Day 12](12/sol_12.aes)

The [twelfth problem](https://adventofcode.com/2019/day/12) was about tracking
objects in 3D, though the physics was a bit simplistic. The naive solution
(after some tweaking) worked ok for part 1. For part 2 a better algorithm was
needed, but still the problem is expensive to solve in Sophia. I think it can
be further optimized, however to get the runtime down to seconds will not be
easily done.

```
16> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/12/sol_12.aes", "solve_1", []).
0 steps / 174491485 gas / 0 reductions / 1993.96ms
{tuple,{7471,
        [{tuple,{{tuple,{-51,-12,-9}},{tuple,{6,8,-9}}}},
         {tuple,{{tuple,{59,-6,-27}},{tuple,{6,1,4}}}},
         {tuple,{{tuple,{21,-60,12}},{tuple,{-20,-9,-4}}}},
         {tuple,{{tuple,{0,81,21}},{tuple,{8,0,9}}}}]}}
20> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/12/sol_12.aes", "solve_2", []).
0 steps / 1652188028015 gas / 0 reductions / 224605.71ms
376243355967784
```
### [Day 13](13/sol_13.aes)

The [thirteenth problem](https://adventofcode.com/2019/day/13) was a game,
namely `Breakout` of course implemented in `Intcode`. Part 1 was simple, run
the program and check the output (the game board). Second part was more fun,
play the game, what is the winning score!? I first tried to do it
interactively, but since you need somewhere around 4500 steps to finish the
game the handling of state trees, etc. killed that approach. Instead I
implemented a simple AI in Sophia and let it play the game to completion. This
took around 90 seconds, but who is in a hurry :-)

```
8> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/13/sol_13.aes", "solve_1", []).
0 steps / 108454225 gas / 0 reductions / 3164.72ms
296
9> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/13/sol_13.aes", "solve_2", []).
0 steps / 87460100420 gas / 0 reductions / 92794.62ms
{tuple,{16,16,16,13824}}
```

### [Day 14](14/sol_14.aes)

The [fourteenth problem](https://adventofcode.com/2019/day/14) was about chains
of "chemical" reactions to create rocket fuel. The main complication was to
keep track of surplus material when reactions where not one-to-one. The task
was to find out how much ore was required to make 1 unit of fuel. For part 2
the problem was reversed, given X ore how much fuel could be produced. Due to
the surplus material it isn't linear, and the best way to approach it was to
implement a binary search over the solution to part 1. I.e. find the amount of
fuel that would require just below X ore. With some small optimizations (only
traverse lists once, etc.) it ran to completion in about 10 seconds.

```
59> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/14/sol_14.aes", "solve_1", []).
0 steps / 762793 gas / 0 reductions / 140.31ms
598038
60> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/14/sol_14.aes", "solve_2", []).
0 steps / 1343915935 gas / 0 reductions / 10722.23ms
2269325
```

### [Day 15](15/sol_15.aes)

The [fifteenth problem](https://adventofcode.com/2019/day/15) was a maze
puzzle. In part 1 you should discover the maze, and find the shortest path to
an object in the maze. In part 2 you should find the point in the maze with the
largest distance to another point. Both problems can be solved in a similar
way: 1) discover the maze, 2) do a breadth-first search over the maze from the
point of interest. This is overkill for part 1, but accidentally I solved part
1 like this (thinking I could optimize later) and that made part 2 trivial!

```
19> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/15/sol_15.aes", "solve_1", []).
0 steps / 1126122416 gas / 0 reductions / 10622.11ms
212
20> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/15/sol_15.aes", "solve_2", []).
0 steps / 1397698224 gas / 0 reductions / 11854.60ms
{tuple,{{tuple,{18,0}},358}}
```

### [Day 16](16/sol_16.aes)

The [sixteenth problem](https://adventofcode.com/2019/day/16) problem was about
flawed frequency transmission. We were tasked with cleaning up a signal (a list
of digits 650 long) where each digit had its own filter and where all filters
should be run 100 times. This translates into a tight loop over a couple of
vectors (or in our case a map). In retrospect this is perhaps the kind of
problem where the disadvantage of being in a VM (optimized for block chain
operations) will be clearly visible. After being at least somewhat clever part
1 of the problem could be solved in about 5 minutes...

Enter part 2, now the signal isn't 650 digits, it is 65000000 digits... But
since we are tasked with finding a chunk of the resulting signal towards the
end (in our case at 5976732) of the signal we could simplify the list of
filters (they all degenerate to the same filter). Armed with this we managed to
(after optimizing in several steps) get the runtime down to a measly 2300
seconds...

```
40> f(M), M = aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/16/sol_16.aes", "solve_1", []).
0 steps / 370908898489 gas / 0 reductions / 282643.28ms
[7,4,3,6,9,0,3,3]
41> f(M), M = aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/16/sol_16.aes", "solve_2", [100]).
0 steps / 164673100419882 gas / 0 reductions / 2296406.90ms
[1,9,9,0,3,8,6,4]
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
