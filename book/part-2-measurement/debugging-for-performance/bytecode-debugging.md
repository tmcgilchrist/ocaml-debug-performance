# Debugging OCaml bytecode

<!-- toc -->

## Preliminaries

### Debuggers

OCaml includes a compiler that produces bytecode and an interpreter for that bytecode.
There are two supported options for debugging bytecode.

1. [ocamldebug](https://v2.ocaml.org/releases/5.1/manual/debugger.html#s%3Ainf-debugger) - provided with the compiler distribution
2. [earlybird](https://github.com/hackwaly/ocamlearlybird) - VSCode integrated debugger using DAP.

Both options reuse the protocol from *ocamldebug* to interface with the bytecode executables.

### DAP

The Debug Adapter Protocol (DAP) defines the abstract protocol used between a development tool (e.g. IDE or editor) and a debugger. The idea behind the Debug Adapter Protocol (DAP) is to abstract the way how the debugging support of development tools communicates with debuggers or runtimes into a protocol. By using DAP OCaml debugging can be integrated with many different IDEs or editors. See [https://microsoft.github.io/debug-adapter-protocol/](https://microsoft.github.io/debug-adapter-protocol/).

[Emacs support for DAP](debugging-dap-emacs.md) using both Bytecode and Native debuggers.

### What is missing?

 * ocamldebug support for DAP
 * No support for MinGW or MSVC Windows ports
 * DAP integration for Vim
 * Limited support for Domains in bytecode debug - single domain only
 * Improvements to [earlybird](https://github.com/hackwaly/ocamlearlybird/issues)