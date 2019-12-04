# Advent of Code - 2019

This repository contains solutions to the [Advent of Code 2019](https://adventofcode.com/2019).
There are many such repositories floating around, the twist in this case is that the solutions
are written in [Sophia](https://github.com/aeternity/protocol/blob/master/contracts/sophia.md)
the smart contract language for the [Aeternity blockchain](https://aeternity.com/).

### [Day 1](01/sol_01.aes)

This year's Advent of Code started off with a straightforward problem. You had to create fuel
consumption for a number of objects depending on their mass. The second part of the problem
also included gas for the mass of the gas required, making the problem (trivially) recursive.

There were no problems writing a [solution](01/sol_01.aes) in Sophia, the recursion is done
with an accumulator to save some gas.

## Running contracts
For obvious reasons I don't want to run these contracts on-chain, and I don't even want to
run them via contract calls etc. Instead we re-use part of the test framework in this way:

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
