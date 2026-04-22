# Execution Profiling

Execution Profiling involves identifying which parts of a program are "hot" (executing frequently enough to impact runtime), causing expensive operations or simply causing "interesting" behaviour (like TLB misses or swapping off-cpu). Identifying this is best done using a profiler.

## Profilers

There are many common profilers that work with OCaml, here we will focus on the following:
 * [perf](https://perfwiki.github.io/main/) is a general-purpose profiler for Linux that uses hardware performance counters. Many visualisation tools like [Hotspot](https://github.com/KDAB/hotspot) and [Firefox Profiler](https://profiler.firefox.com/) take perf output.
 * [Instruments](https://developer.apple.com/xcode/) is a general purpose profiler provided with XCode on macOS. It uses Dtrace technology that is built into macOS and provides custom scripting of probes.
 * [pmcstat](https://man.freebsd.org/cgi/man.cgi?query=pmcstat&sektion=8) is a performance measurement tool for FreeBSD that uses hardware performance counters, similar to "perf".
 * [dtrace]()
 * [samply](https://github.com/mstange/samply/) is a sampling profiler that produces profiles that can be viewed in the Firefox Profiler. It works on Mac, Linux, and Windows.
 * flamegraph is a Cargo command that uses perf/DTrace to profile your code and then displays the results in a flame graph. It works on Linux and all platforms that support DTrace (macOS, FreeBSD, and possibly Illumos).

## Debug Info

To profile a release build you should enable debug information. To do this, add "-g" to your flags in dune:

```
(env
 (dev TODO What is the difference between flags and ocamlopt_flags?
   (flags (:standard -g))
   (ocamlopt_flags (:standard -g)))
 (release
  (ocamlopt_flags (:standard -g))))
```

where `dev` and `release` correspond to build profiles. Alternatively add "-g" to the ocamlopt native compiler.

TODO How can we force "-g" for all packages in an opam switch including the OCaml compiler itself?

At the time of writing this will include Call Frame Information (CFI) suitable for unwinding the call stack and source line debug information (identifying how symbols in a binary map back to source code). Ensure that no build tools call `strip` on the output binaries, this will remove debug information.

## Frame Pointers



## Profiling OCaml with perf

 * perf support for DWARF vs FP vs LBR
 * Importance of DWARF for performance tools like perf

Take content from [ocaml/ocaml#12563](https://github.com/ocaml/ocaml/issues/12563),
[ocaml/ocaml#11144](https://github.com/ocaml/ocaml/pull/11144)
and [ocaml/ocaml#11031](https://github.com/ocaml/ocaml/pull/11031)

LBR descriptions [https://lwn.net/Articles/680985/](https://lwn.net/Articles/680985/)
and [https://lwn.net/Articles/680996/](https://lwn.net/Articles/680996/).

perf is the official Linux profiler. It is a large tool that covers many areas of profiling, tracing and scripting.
Here we will only cover the intersection between OCaml and perf, and will leave further discussion to other books like [Systems Performance by Brendan Gregg] which will do a more thorough job of it.

## Profiling with Instruments

## Profiling with 