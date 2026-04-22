# Real-World Optimisation Examples

<!-- toc -->

This chapter will contain detailed case studies of real-world OCaml performance optimisation work. Suggested topics include:

- **Optimising solver service** - Investigating the scaling performance of the solving service used in OCaml CI infrastructure.

- **Static Analysis performance** - Tuning the Infer static analysis service for better OCaml multicore performance. Covering perf CPU and off-CPU profiling, Garbage Collection behaviour and memory usage.

- **Web Server Performance** - Tuning an OCaml web server (e.g., Dream, Cohttp). Connection handling, request parsing, response serialisation, and Eio integration for concurrent I/O.

- **JSON Parsing: A Microbenchmark Study** - Comparing approaches to JSON parsing in OCaml (Yojson, Jsonm, custom parsers). Allocation profiles, buffer management, and streaming vs tree parsing.

Contributions and suggestions for case studies are welcome.
