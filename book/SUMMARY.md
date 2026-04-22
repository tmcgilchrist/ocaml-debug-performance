# The OCaml Performance Book

[Preface](preface.md)

---

# Part I: Preliminaries

- [Preliminaries](part-1-preliminaries/README.md)
  - [How to Use This Book](part-1-preliminaries/introduction/how-to-use-this-book.md)
  - [Who This Book Is For](part-1-preliminaries/introduction/who-this-book-is-for.md)
  - [What Makes OCaml Fast](part-1-preliminaries/introduction/what-makes-ocaml-fast.md)
  - [The Observability Stack](part-1-preliminaries/introduction/observability-stack.md)
- [Philosophy and Methodology](part-1-preliminaries/philosophy-and-methodology/measure-first.md)
  - [The USE Method for OCaml](part-1-preliminaries/philosophy-and-methodology/use-method.md)
  - [The Performance Triage Process](part-1-preliminaries/philosophy-and-methodology/performance-triage.md)
  - [Setting Up Reproducible Benchmarks](part-1-preliminaries/philosophy-and-methodology/reproducible-benchmarks.md)
  - [The Performance Checklist](part-1-preliminaries/philosophy-and-methodology/performance-checklist.md)
- [Build Configuration for Performance](part-1-preliminaries/build-configuration/dune-profiles.md)
  - [Compiler Optimization Levels](part-1-preliminaries/build-configuration/compiler-optimization-levels.md)
  - [Flambda: The Optimizing Backend](part-1-preliminaries/build-configuration/flambda.md)
  - [Native Code vs Bytecode](part-1-preliminaries/build-configuration/native-vs-bytecode.md)

---

# Part II: Measurement, Profiling, and Observation

- [Profiling and Observability Tools Overview](part-2-measurement/README.md)
  - [The Observability Landscape](part-2-measurement/observability-tools/observability-landscape.md)
  - [Tool Categories for OCaml](part-2-measurement/observability-tools/tool-categories.md)
  - [Choosing the Right Tool](part-2-measurement/observability-tools/choosing-the-right-tool.md)
  - [Profiling Ecosystem](part-2-measurement/observability-tools/profiling.md)
- [CPU Profiling](part-2-measurement/cpu-profiling/using-perf.md)
  - [Generating Flamegraphs](part-2-measurement/cpu-profiling/flamegraphs.md)
    - [Flamegraphs on Linux](part-2-measurement/cpu-profiling/flamegraphs-linux.md)
    - [Flamegraphs on macOS](part-2-measurement/cpu-profiling/flamegraphs-macos.md)
  - [Using Instruments on macOS](part-2-measurement/cpu-profiling/instruments-macos.md)
  - [Time-based Profiling with Landmarks](part-2-measurement/cpu-profiling/landmarks.md)
- [Dynamic Tracing and USDT Probes](part-2-measurement/dynamic-tracing/introduction-to-dynamic-tracing.md)
  - [OCaml Runtime USDT Probes](part-2-measurement/dynamic-tracing/runtime-usdt-probes.md)
  - [Application-Level USDT](part-2-measurement/dynamic-tracing/application-usdt.md)
  - [Tracing Workflows](part-2-measurement/dynamic-tracing/tracing-workflows.md)
  - [Building Custom Analysis Tools](part-2-measurement/dynamic-tracing/custom-analysis-tools.md)
- [Memory Profiling](part-2-measurement/memory-profiling/memory-layout.md)
  - [Statistical Memory Profiling with statmemprof](part-2-measurement/memory-profiling/statmemprof-4-lts.md)
    - [statmemprof in OCaml 5.x](part-2-measurement/memory-profiling/statmemprof-5.md)
  - [Using memtrace and memtrace-viewer](part-2-measurement/memory-profiling/memtrace.md)
  - [Other Memory Tools](part-2-measurement/memory-profiling/other-memory-tools.md)
  - [GC Statistics and Tuning](part-2-measurement/memory-profiling/gc-statistics-and-tuning.md)
  - [Combining Memory Profiling with USDT](part-2-measurement/memory-profiling/memory-profiling-with-usdt.md)
- [Debugging for Performance](part-2-measurement/debugging-for-performance/native-debugging.md)
  - [GDB on Linux Debugging Session](part-2-measurement/debugging-for-performance/sample-debugging-session-gdb.md)
  - [LLDB on macOS Debugging Session](part-2-measurement/debugging-for-performance/sample-debugging-session-lldb.md)
  - [Bytecode Debugging](part-2-measurement/debugging-for-performance/bytecode-debugging.md)
    - [Debugging OCaml with Emacs DAP](part-2-measurement/debugging-for-performance/debugging-dap-emacs.md)
  - [Inspecting Compiler Output](part-2-measurement/debugging-for-performance/inspecting-compiler-output.md)
- [Benchmarking](part-2-measurement/benchmarking/micro-benchmarking.md)
  - [Macro-benchmarking Strategies](part-2-measurement/benchmarking/macro-benchmarking.md)
  - [Continuous Benchmarking in CI](part-2-measurement/benchmarking/continuous-benchmarking.md)

---

# Part III: Optimizations

- [Compiler-Level Optimizations](part-3-optimizations/README.md)
  - [Inlining](part-3-optimizations/compiler-optimizations/inlining.md)
  - [Unboxing and Specialization](part-3-optimizations/compiler-optimizations/unboxing-and-specialization.md)
  - [Tail Call Optimization](part-3-optimizations/compiler-optimizations/tail-call-optimization.md)
- [Data Structure Choices](part-3-optimizations/data-structures/lists-arrays-sequences.md)
  - [Hash Tables and Maps](part-3-optimizations/data-structures/hash-tables-and-maps.md)
  - [Strings and Buffers](part-3-optimizations/data-structures/strings-and-buffers.md)
  - [Records and Tuples](part-3-optimizations/data-structures/records-and-tuples.md)
  - [Custom Data Structures](part-3-optimizations/data-structures/custom-data-structures.md)
- [Memory and Allocation](part-3-optimizations/memory-and-allocation/reducing-allocations.md)
  - [Boxing and Unboxing](part-3-optimizations/memory-and-allocation/boxing-and-unboxing.md)
  - [GC-Friendly Programming](part-3-optimizations/memory-and-allocation/gc-friendly-programming.md)
  - [Memory Layout Optimization](part-3-optimizations/memory-and-allocation/memory-layout-optimization.md)
- [Function Optimization](part-3-optimizations/function-optimization/closures.md)
  - [Partial Application](part-3-optimizations/function-optimization/partial-application.md)
  - [Higher-Order Functions](part-3-optimizations/function-optimization/higher-order-functions.md)
  - [Exception Handling](part-3-optimizations/function-optimization/exception-handling.md)
- [I/O Performance](part-3-optimizations/io-performance/file-io.md)
  - [Network I/O](part-3-optimizations/io-performance/network-io.md)
  - [Serialization](part-3-optimizations/io-performance/serialization.md)

---

# Part IV: Parallelism and Concurrency (OCaml 5)

- [Understanding OCaml 5's Parallelism Model](part-4-parallelism/README.md)
  - [Domains: True Parallelism](part-4-parallelism/parallelism-model/domains.md)
  - [The Multicore GC](part-4-parallelism/parallelism-model/multicore-gc.md)
  - [Effects for Concurrency](part-4-parallelism/parallelism-model/effects.md)
  - [Observing Multicore Behavior](part-4-parallelism/parallelism-model/observing-multicore.md)
- [Parallel Programming with Domainslib](part-4-parallelism/domainslib/task-pools.md)
  - [Async/Await Patterns](part-4-parallelism/domainslib/async-await.md)
  - [Channels and Communication](part-4-parallelism/domainslib/channels.md)
  - [Common Pitfalls](part-4-parallelism/domainslib/common-pitfalls.md)
- [Concurrent I/O with Eio](part-4-parallelism/eio/direct-style-io.md)
  - [Fiber Scheduling](part-4-parallelism/eio/fiber-scheduling.md)
  - [Combining Eio and Domainslib](part-4-parallelism/eio/eio-and-domainslib.md)
  - [Performance Tuning](part-4-parallelism/eio/performance-tuning.md)
- [Memory and Synchronization](part-4-parallelism/memory-and-synchronization/atomic-operations.md)
  - [Lock-Free Data Structures](part-4-parallelism/memory-and-synchronization/lock-free-data-structures.md)
  - [Avoiding Contention](part-4-parallelism/memory-and-synchronization/avoiding-contention.md)
  - [Cache Coherency Costs](part-4-parallelism/memory-and-synchronization/cache-coherency.md)
- [Migrating from OCaml 4 to 5](part-4-parallelism/migrating-ocaml-4-to-5/single-threaded-comparison.md)
  - [Adapting Multi-process Code](part-4-parallelism/migrating-ocaml-4-to-5/adapting-multi-process.md)
  - [Common Migration Issues](part-4-parallelism/migrating-ocaml-4-to-5/common-migration-issues.md)

---

# Part V: Interoperability

- [C Bindings Performance](part-5-interoperability/README.md)
  - [The Foreign Function Interface](part-5-interoperability/c-bindings/ffi.md)
  - [Minimizing Crossing Overhead](part-5-interoperability/c-bindings/minimizing-overhead.md)
  - [Callback Costs](part-5-interoperability/c-bindings/callback-costs.md)
  - [Memory Management Across Boundaries](part-5-interoperability/c-bindings/memory-across-boundaries.md)
  - [ctypes vs Hand-Written Stubs](part-5-interoperability/c-bindings/ctypes-vs-stubs.md)
- [JavaScript Compilation](part-5-interoperability/javascript/jsoo-performance.md)
  - [Optimization Flags](part-5-interoperability/javascript/optimization-flags.md)
  - [Dead Code Elimination](part-5-interoperability/javascript/dead-code-elimination.md)

---

# Part VI: Compile Times

- [Reducing Compilation Time](part-6-compile-times/README.md)
  - [Understanding What's Slow](part-6-compile-times/reducing-compilation-time/understanding-whats-slow.md)
  - [Module Structure for Fast Builds](part-6-compile-times/reducing-compilation-time/module-structure.md)
  - [Flambda Compile Time Trade-offs](part-6-compile-times/reducing-compilation-time/flambda-compile-times.md)
  - [Parallel Compilation](part-6-compile-times/reducing-compilation-time/parallel-compilation.md)
  - [Incremental Builds and Caching](part-6-compile-times/reducing-compilation-time/incremental-builds.md)

---

# Part VII: Case Studies

- [Real-World Optimization Examples](part-7-case-studies/README.md)
  - [Case Studies](part-7-case-studies/real-world-examples/real-world-examples.md)
- [Common Performance Patterns](part-7-case-studies/common-patterns/defunctionalization.md)
  - [Continuation-Passing for Tail Calls](part-7-case-studies/common-patterns/continuation-passing.md)
  - [Memoization Strategies](part-7-case-studies/common-patterns/memoization.md)
  - [Lazy Evaluation Trade-offs](part-7-case-studies/common-patterns/lazy-evaluation.md)
  - [Data-Oriented Design in OCaml](part-7-case-studies/common-patterns/data-oriented-design.md)

---

# Appendices

- [Quick Reference: Compiler Flags](appendices/compiler-flags.md)
- [Quick Reference: Dune Configuration](appendices/dune-configuration.md)
- [Tool Installation Guide](appendices/tool-installation.md)
- [Glossary](appendices/glossary.md)
- [Further Reading](appendices/further-reading.md)
