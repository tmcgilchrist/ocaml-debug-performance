# Profiling OCaml with perf

 * perf support for DWARF vs FP vs LBR
 * Importance of DWARF for performance tools like perf

Take content from [ocaml/ocaml#12563](https://github.com/ocaml/ocaml/issues/12563),
[ocaml/ocaml#11144](https://github.com/ocaml/ocaml/pull/11144)
and [ocaml/ocaml#11031](https://github.com/ocaml/ocaml/pull/11031)

LBR descriptions [https://lwn.net/Articles/680985/](https://lwn.net/Articles/680985/)
and [https://lwn.net/Articles/680996/](https://lwn.net/Articles/680996/).

perf is the official Linux profiler. It is a large tool that covers many areas of profiling, tracing and scripting.
Here we will only cover the intersection between OCaml and perf, and will leave further discussion to other books like [Systems Performance by Brendan Gregg] which will do a more thorough job of it.

