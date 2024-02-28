# Debugging support in the OCaml compiler

<!-- toc -->

This document explains the state of debugging tools support in the OCaml compiler.
It gives an overview of GDB, LLDB, WinDbg/CDB, as well as infrastructure around OCaml
compiler to debug OCaml code.

The material contains both current support and ideas for future areas to improve.

## Preliminaries

### Debuggers

According to wikipedia [1]

> A debugger or debugging tool is a computer program used to test and debug other programs (the "target" program).

Writing a debugger from scratch for a language requries considerable work, especially if you want to support
various platforms like Linux, MacOS, and Windows. Existing debuggers like GDB and LLDB can be extended to support
debugging a language like OCaml. This is the path OCaml has chosen for what I'll call native debugging, debugging
exectuables compiled to object code via assembly and run on a CPU. OCaml also includes a debugger for the bytecode
exectuables it also produces, more on that later.

### DWARF

According to the [DWARF] standard website:

> DWARF is a debugging information file format used by many compilers and debuggers to support source level
> debugging. It addresses the requirements of a number of procedural languages, such as C, C++, and Fortran,
> and is designed to be extensible to other languages. DWARF is architecture independent and applicable to
> any processor or operating system. It is widely used on Unix, Linux and other operating systems, as well
> as in stand-alone environments.

DWARF allows a compiler to describe how program source translates to assembly, using a data structure called
Debugging Information Entry (DIE) which stores the information as "tags" to denote functions, variables etc.,
e.g., `DW_TAG_variable`, `DW_TAG_pointer_type`, `DW_TAG_subprogram` etc.
You can also invent your own tags and attributes.

DWARF reader is a program that consumes the DWARF format and creates debugger compatible output.
This program may live in the compiler itself.

### CodeView/PDB

[PDB] (Program Database) is a file format created by Microsoft that contains debug information.
PDBs can be consumed by debuggers such as WinDbg/CDB and other tools to display debug information.
A PDB contains multiple streams that describe debug information about a specific binary such
as types, symbols, and source files used to compile the given binary. CodeView is another
format which defines the structure of [symbol records] and [type records] that appear within
PDB streams.

### Compact Unwinding Format

Apple introduced a new kind of unwinding info the “compact unwinding format” on Apple platforms like macOS and iOS.
The Clang compiler on those platforms emits this format along with DWARF CFI. The format is described by the implementation
in clang/llvm, with an independent description provided at [https://faultlore.com/blah/compact-unwinding/](https://faultlore.com/blah/compact-unwinding/) and [https://github.com/mstange/macho-unwind-info](https://github.com/mstange/macho-unwind-info) So to generate good backtraces on Apple platforms, you need to be able to parse and interpret compact unwinding tables.

## Supported debuggers
### GDB

OCaml supports GBD on Linux for all OCaml supported platforms, it supports basic debugging like setting breakpoints, stepping,
backtraces etc. A sample GDB/Linux session is [shown here](sample-debugging-session.md).
Additionally there are GDB macros [gdb-macros](https://github.com/ocaml/ocaml/blob/trunk/tools/gdb-macros) and Python macros [gdb_ocamlrun.py](https://github.com/ocaml/ocaml/blob/trunk/tools/gdb_ocamlrun.py) that provide low-level debugging of OCaml programs and of the
 OCaml runtime itself (both native and byte-code).

#### OCaml parser extensions

To be able to show debug output, we need an expression parser. GDB expression parsers are written in [Bison],
and could be written to accept a subset of OCaml expressions, including printing OCaml values knowing the
boxed types used by OCaml. Read the [Memory Representation of Values] chapter of RealWorld OCaml for more details.

Future work:

 * Restore DWARF CFI for POWER native code
 * Port more GDB macros to Python
 * Add a source language to GDB https://sourceware.org/gdb/wiki/Internals%20Adding-a-Source-Language-to-GDB

### LLDB

LLDB is the default debugger on MacOS, shipped with XCode and generally works the best on that platform.
It is also available on Linux and is the [default debugger for FreeBSD].

 * LLDB has a plugin architecture but that does not work for language support.
 * At present GDB generally works better on Linux.

#### OCaml parser extensions
This expression parser is written in C++. It is a type of Recursive Descent parser.

Some initial work on OCaml LLDB support at https://github.com/ocaml-flambda/llvm-project

**What is included here?**

### RR

rr is a lightweight tool for recording, replaying and debugging execution of applications (trees of processes and threads).
Debugging extends gdb with very efficient reverse-execution, which in combination with standard gdb/x86 features like
hardware data watchpoints, makes debugging much more fun. OCaml supports RR on Linux for certain x86_64 and ARM64 platforms.

Future work:
 * RR doesn't currently support Apple M3 chips see https://github.com/rr-debugger/rr/pull/3528

### WinDbg/CDB

Microsoft provides [Windows Debugging Tools] such as the Windows Debugger (WinDbg) and the Console Debugger (CDB)
which both support debugging programs that provide PDB information. These debuggers parse the debug info for a
binary from the PDB, if available, to construct a visualization to serve up in the debugger. Currently the OCaml
compiler does not produce PDB and future work is required to support debugging on Windows. Additionally OCaml on Windows
provides three different ports, complicating the matter further.


## DWARF and OCaml

* Importance of DWARF for performance tools like perf
* Provide a small example of how OCaml gets mapped to DWARF DIEs

DW_TAG_base_type provide base type mappings that can be defined per language.
Should OCaml be using this to output DWARF representations.

DW_TAG_variable describes a variable in the source language

https://dwarfstd.org/doc/Debugging%20using%20DWARF-2012.pdf

```
$ 
COMPILE_UNIT<header overall offset = 0x00000047>:
< 0><0x0000000b>  DW_TAG_compile_unit
                    DW_AT_stmt_list             0x00000062
                    DW_AT_low_pc                0x0004b6a0
                    DW_AT_high_pc               0x0004b760
                    DW_AT_name                  fib.ml
                    DW_AT_comp_dir              /home/tsmc/ocaml
                    DW_AT_producer              GNU AS 2.41
                    DW_AT_language              DW_LANG_Mips_Assembler

LOCAL_SYMBOLS:
< 1><0x0000002e>    DW_TAG_subprogram
                      DW_AT_name                  camlFib__main_1_3_code
                      DW_AT_external              yes(1)
                      DW_AT_type                  <0x00000073>
                      DW_AT_low_pc                0x0004b6f8
                      DW_AT_high_pc               0x0004b73c
< 1><0x00000045>    DW_TAG_subprogram
                      DW_AT_name                  camlFib__fib_0_2_code
                      DW_AT_external              yes(1)
                      DW_AT_type                  <0x00000073>
                      DW_AT_low_pc                0x0004b6a0
                      DW_AT_high_pc               0x0004b6f4
< 1><0x0000005c>    DW_TAG_subprogram
                      DW_AT_name                  camlFib__entry
                      DW_AT_external              yes(1)
                      DW_AT_type                  <0x00000073>
                      DW_AT_low_pc                0x0004b740
                      DW_AT_high_pc               0x0004b760
< 1><0x00000073>    DW_TAG_unspecified_type
```

## OCaml Name Mangling

## What is missing?

## Resources

Tom Tromey discusses debugging support in rustc, provides a good overview of the area and what OCaml
might also want to do - [https://www.youtube.com/watch?v=elBxMRSNYr4](https://www.youtube.com/watch?v=elBxMRSNYr4)

[https://rustc-dev-guide.rust-lang.org/debugging-support-in-rustc.html](https://rustc-dev-guide.rust-lang.org/debugging-support-in-rustc.html)

DWARF support in GHC (4 part series) [https://well-typed.com/blog/2020/04/dwarf-1/](https://well-typed.com/blog/2020/04/dwarf-1/)


[1]: https://en.wikipedia.org/wiki/Debugger
[DWARF]: http://dwarfstd.org
[PDB]: https://llvm.org/docs/PDB/index.html
[symbol records]: https://llvm.org/docs/PDB/CodeViewSymbols.html
[type records]: https://llvm.org/docs/PDB/CodeViewTypes.html
[default debugger for FreeBSD]: https://docs.freebsd.org/en/books/developers-handbook/tools/#debugging
[Memory Representation of Values]: https://dev.realworldocaml.org/runtime-memory-layout.html
[Windows Debugging Tools]: https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/