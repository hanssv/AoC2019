# Advent of Code - 2019

This repository contains solutions to the [Advent of Code
2019](https://adventofcode.com/2019). There are many such repositories floating
around, the twist in this case is that the solutions are written in
[Sophia](https://github.com/aeternity/protocol/blob/master/contracts/sophia.md)
the smart contract language for the [Aeternity
blockchain](https://aeternity.com/).

## Update 2020-01-16 `@compiler >= 4.2.0`

Below are small notes per day. But following the release of the new Sophia
compiler (v4.2.0) I also took the time to update the solutions with the new
syntax (separate entrypoint/function type signature and definition, and pattern
matching in left-hand sides. The changes are mostly cosmetic, but also in some
cases the cleaner code allows for more optimizations by the compiler. The
compiler update is fully backwards compatible, so these changes are only
optimal.

For example this allows us to rewrite:
```Sophia
  function cmp(p, q) =
    switch((p, q))
      ((x1, y1), (x2, y2)) =>
        (abs(x1) + abs(y1)) =< (abs(x2) + abs(y2))
```
into:
```Sophia
  function cmp((x1, y1), (x2, y2)) = abs(x1) + abs(y1) =< abs(x2) + abs(y2)
```

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

The [sixteenth problem](https://adventofcode.com/2019/day/16) was about
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

### [Day 17](17/sol_17.aes)

The [seventeenth problem](https://adventofcode.com/2019/day/17) was
(there is an every second day pattern...) another `Intcode` problem. The task
was to explore a maze (or a bunch of scaffolding), in part 1 we should compute
a sum of all coordinates that was an intersection - entirely straightforward.

Part 2 was about providing a program to a small robot that should visit all of
the maze, the caveat was that there was only a limited amount of space for the
program (including sub-routines). Because of the construction of the maze it
was, however, easy to write a navigation function that visited all of the maze,
and then break the resulting trace up into suitable sub-routines. I ended up
only solving this for my input, not the general case. No problems with run-time
today.

```
122> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/17/sol_17.aes", "setup", []).
0 steps / 265526634 gas / 0 reductions / 5207.16ms
Store:
  #{1 =>
        {tuple,{#{{tuple,{16,22}} => {tuple,{}},
                  {tuple,{14,13}} => {tuple,{}},
                  ...},
                {tuple,{6,0}}}}}
{tuple,{}}
123> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/17/sol_17.aes", "solve_1", []).
0 steps / 71673 gas / 0 reductions / 24.34ms
5788
124> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/17/sol_17.aes", "solve_2", []).
0 steps / 1199125790 gas / 0 reductions / 11443.52ms
648545
```

### [Day 18](18/sol_18.aes)

The [eighteenth problem](https://adventofcode.com/2019/day/18) was a
shortest path finding problem in a maze with locked doors. This makes a really
tricky problem, since the possible paths keeps changing once you make your way
through the maze. I managed to solve the problem, using Erlang, but that
solution has to be improved by an order of magnitude (or two!) to be feasible
to run in Sophia :-(

~~So temporarily I admit defeat, but I haven't given up entirely yet... I think
it can be done.~~ Yes, it could be done. Re-working the algorithm to
pre-compute a graph of possible moves, a clever heap implementation for the
shortest path algorithm and using bit-vectors to represent the list of keys all
together makes this runnable in Sophia. The credit for this cleverness goes to
[Ulf](https://github.com/UlfNorell)!

```
82> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/18/sol_18.aes", "setup_1", []).
0 steps / 300095817507 gas / 0 reductions / 156651.49ms
Store:
  #{1 =>
        #{0 =>
              [{tuple,{5,162,{bits,0},{bits,4210944}}},
               {tuple,{9,330,{bits,69206016},{bits,526356}}},
               ...
{tuple,{}}
83> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/18/sol_18.aes", "solve_1", []).
0 steps / 25350509087 gas / 0 reductions / 298800.33ms
{tuple,{[22,14,8,7,25,17,5,1,2,11,26,21,15,20,4,19,16,23,9,
         10,6,3,24,12,18,13],
        4520}}
85> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/18/sol_18.aes", "setup_2", []).
0 steps / 7874835496 gas / 0 reductions / 22853.58ms
Store:
  #{1 =>
        #{-3 =>
              [{tuple,{9,328,{bits,69206016},{bits,526356}}},
               {tuple,{16,346,{bits,69206016},{bits,526340}}},
               ...
{tuple,{}}
86> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/18/sol_18.aes", "solve_2", []).
0 steps / 69303116775 gas / 0 reductions / 389509.87ms
{tuple,{[11,24,26,8,22,14,1,2,21,3,15,5,7,17,25,6,20,19,16,
         4,9,23,10,12,18,13],
        1540}}
```

### [Day 19](19/sol_19.aes)

The [nineteenth problem](https://adventofcode.com/2019/day/19) was
about an `Intcode` program (of course). The program described a beam in 2D
space. For part 1 you had to run the program 2500 times (or you could be
clever, but since this was entirely feasible in minutes, why bother!). For part
2 you had to trace the outline of the beam but again nice and easy compared to
yesterday!

```
64> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/19/sol_19.aes", "solve_1", []).
0 steps / 175755223649 gas / 0 reductions / 130745.37ms
141
65> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/19/sol_19.aes", "solve_2", []).
0 steps / 53045376977 gas / 0 reductions / 106968.82ms
{tuple,{1564,1348,15641348}}
```

### [Day 20](20/sol_20.aes)

The [twentieth problem](https://adventofcode.com/2019/day/20) was yet another
maze. Part of the problem was to parse the input, but since we still lack some
string functions in Sophia we did some pre-processing. Once in place part 1 was
a simple breadth-first search. Part 2 was a bit more tricky, where we were to
visit several instances of the maze.

```
19> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/20/sol_20.aes", "solve_1", []).
0 steps / 218213962 gas / 0 reductions / 1425.99ms
654
22> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/20/sol_20.aes", "solve_2", []).
0 steps / 140855720130 gas / 0 reductions / 36148.79ms
7360
```

### [Day 21](21/sol_21.aes)

The [twenty-first problem](https://adventofcode.com/2019/day/21) was about
writing a little bit of assembly-like code to solve a puzzle. Nothing
algorithmic but instead a straight-forward puzzle. Part 1 was quite easy. The
twist for part 2 made it a bit harder but not overly so. Run-time in Sophia was
a few seconds for part 1 and a minute and a half for part 2. The Sophia
`Intcode` interpreter isn't super-fast...

```
43> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/21/sol_21.aes", "solve_1", []).
0 steps / 173980099 gas / 0 reductions / 4115.54ms
19354083
44> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/21/sol_21.aes", "solve_2", []).
0 steps / 86713494597 gas / 0 reductions / 96768.97ms
1143499964
```

### [Day 22](22/sol_22.aes)

The [twenty-second problem](https://adventofcode.com/2019/day/22) was about
shuffling a (huge) deck of cards. For part 1 you could get away with a naive
implementation that actually did handle the deck of cards. But the run-time was
not impressive (minutes in Sophia). To optimize you should figure out that the
solution only asks for a single card (and hence a single index) in the deck. So
we can transform the problem to only track one index at a time.

For part 2 where the deck isn't huge but enormous (containing around 10^14
cards) this is crucial, and we should also perform the shuffling no less than
101741582076661 times. I.e. no naive solution will cut it. The key observation
to make is that all different shuffle techniques are linear, i.e. we turn the
sequence of shuffle-steps into a single function `ax + b = y`. So the problem
reduces to finding `a` and `b` and then apply it to `101741582076661`. And all
of this should be done modulo the deck size (a large prime). Lots of things to
get wrong, but once the bits fall into place the code turns out nicely and it
runs quickly.

```
53> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/22/sol_22.aes", "solve_1", []).
0 steps / 22218 gas / 0 reductions / 7.65ms
7860
54> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/22/sol_22.aes", "solve_2", []).
0 steps / 205431 gas / 0 reductions / 43.75ms
61256063148970
```

### [Day 23](23/sol_23.aes)

The [twenty-third problem](https://adventofcode.com/2019/day/23) was about
running NICs (in `Intcode`) and simulate a network. Part 1 was quick, just
create 50 `Intcode` processes and then take care of routing messages. Finally,
abort once a packet is sent to the NAT controller (address 255).

For part 2 we should also detect idleness in the network, I didn't do anything
fancy just once a full cycle without a packet occurs, decide the network is
idle. Keep track of NAT messages and halt once it cycles.

```
7>  aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/23/sol_23.aes", "solve_1", []).
0 steps / 32864407 gas / 0 reductions / 1562.02ms
{revert,<<"Addr 255 is not routeable: (47969,17283)">>}
8>  aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/23/sol_23.aes", "solve_2", []).
0 steps / 8909324928 gas / 0 reductions / 29257.81ms
11319
```

### [Day 24](24/sol_24.aes)

The [twenty-fourth problem](https://adventofcode.com/2019/day/24) was about a
variant of game of life. I solved part 1 using Bit-arrays (`bits`) since we had
to score the first repeated game board and that score was exactly the bit array
:-) The twist for part 2 was to make the board infinitely recursive - though we
only had to make 200 iterations so the size was manageable. Since it is
Christmas I didn't have time to optimize it, one could limit the computation
quite a bit rather easily. But it runs in about 5 minutes.

```
162> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/24/sol_24.aes", "solve_1", []).
0 steps / 1600333 gas / 0 reductions / 241.96ms
{bits,17321586}
163> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/24/sol_24.aes", "solve_2", [200]).
0 steps / 2572446815615 gas / 0 reductions / 273581.75ms
1921
```

### [Day 25](25/sol_25.aes)

The [twenty-fifth problem](https://adventofcode.com/2019/day/25) was an
`Intcode` text adventure! Since interaction is slow in a text adventure we keep
the state in the contract and call it for each command. Today there was only a
part one, and to keep the description reasonably short I've fused the commands
necessary to complete the puzzle into just two command sequences. I also
defined the `Cmd` help function in the Erlang shell.

So, success! It could be done, and most days the extra challenge of doing it in
Sophia wasn't too bad. Some days were challenging, but all of them got solved.
In the meantime we collected lots of ideas for improvements of Sophia, some
already implemented and not released and some more in the pipeline!

```
42> f(Cmd), Cmd = fun(C) -> S = aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/25/sol_25.aes", "run", [C], [no_store]), io:format("~s", [S]) end.                     #Fun<erl_eval.6.99386804>
43> Cmd = fun(C) -> S = aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/25/sol_25.aes", "run", [C], [no_store]), io:format("~s", [S]) end.
#Fun<erl_eval.6.99386804>
44> aefa_sophia_test:run_file("/Users/hans/Personal/Repos/AoC2019/25/sol_25.aes", "init", [], [no_store]).                                                                           0 steps / 813523 gas / 0 reductions / 224.86ms
{tuple,{}}
45> Cmd("west\ntake hypercube\nwest\nwest\nnorth\ntake shell\nwest\n\nsouth\ntake festive hat\nnorth\neast\nsouth\neast\neast\neast\n").                                             0 steps / 949057396 gas / 0 reductions / 32923.85ms



== Hull Breach ==
You got in through a hole in the floor here. To keep your ship from also freezing, the hole has been sealed.

...
Command?
ok
46>  Cmd("east\nnorth\nwest\nnorth\nwest\nwest\ntake astronaut ice cream\nsouth\nsouth\n").
0 steps / 241315500 gas / 0 reductions / 13948.85ms



== Crew Quarters ==
The beds are all too small for you.

...
== Pressure-Sensitive Floor ==
Analyzing...

Doors here lead:
- north

A loud, robotic voice says "Analysis complete! You may proceed." and you enter the cockpit.
Santa notices your small droid, looks puzzled for a moment, realizes what has happened, and radios your ship directly.
"Oh, hello! You should be able to get in by typing 33624080 on the keypad at the main airlock."
ok
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

### Aeternity public key
`ak_GTWqTvWENrEZ44RaT4S48jER6vSRBFiEMnJgf78rBShSLVUhA`
