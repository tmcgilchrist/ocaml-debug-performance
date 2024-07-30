# Sample LLDB on MacOS session

Starting from an OCaml 4.14 switch, create one if it doesn't already exist with `opam switch create 4.14.1 --no-install`.

``` shell
$ opam switch
#  switch                                              compiler                    description
   4.14.1                                              ocaml-base-compiler.4.14.1  4.14.1
```

Consider this program:

``` ocaml
$ cat fib.ml
let rec fib n =
  if n < 2 then 1
	else fib (n-1) + fib (n-2)

let main () =
  let r = fib 20 in
	Printf.printf "fib(20) = %d" r

let _ = main ()
```

compiled with

``` ocaml
$ ocamlopt -g -o fib-4.14.1.exe fib.ml
```

Here the OCaml from our fib program gets name mangled into the following:

``` ocaml
$ nm -pa fib-4.14.1.exe|grep "camlFib"
000000010005da58 D _camlFib
000000010005daf0 D _camlFib__1
000000010005dac8 D _camlFib__2
000000010005dab0 D _camlFib__3
000000010005da98 D _camlFib__4
000000010005da80 D _camlFib__5
000000010005da40 D _camlFib__6
000000010005da28 D _camlFib__7
0000000100003838 T _camlFib__code_begin
0000000100003928 T _camlFib__code_end
000000010005da20 D _camlFib__data_begin
000000010005db08 D _camlFib__data_end
00000001000038e8 T _camlFib__entry
0000000100003838 T _camlFib__fib_267
000000010005db10 D _camlFib__frametable
000000010005da68 D _camlFib__gc_roots
0000000100003890 T _camlFib__main_269
```

OCaml functions are mangled as `caml<MODULENAME>__<FUNCTIONNAME>_<RANDOMINT>`. The numbers used can be recovered from the lambda format. Re-running the command with `-dlambda` will output the lamdba form, and `-S` will output the assembly for the program as `fib.S`. You can see the symbol `_camlFib__main_269` is coming from the `main/269` seen in the lambda format.

``` ocaml
$ ocamlopt -dlambda -g -S -o fib-4.14.1.exe fib.ml
(seq
  (letrec
    (fib/267
       (function n/268[int] : int
         (if (< n/268 2) 1
           (+ (apply fib/267 (- n/268 1)) (apply fib/267 (- n/268 2))))))
    (setfield_ptr(root-init) 0 (global Fib!) fib/267))
  (let
    (main/269 =
       (function param/308[int] : int
         (let (r/271 =[int] (apply (field 0 (global Fib!)) 20))
           (apply (field 1 (global Stdlib__Printf!))
             [0: [11: "fib(20) = " [4: 0 0 0 0]] "fib(20) = %d"] r/271))))
    (setfield_ptr(root-init) 1 (global Fib!) main/269))
  (apply (field 1 (global Fib!)) 0) 0 0)
```

``` ocaml
 $ lldb fib-4.14.1.exe 
(lldb) target create "fib-4.14.1.exe"
Current executable set to '/Users/tsmc/projects/ocaml/fib-4.14.1.exe' (arm64).
(lldb) br s -n camlFib__fib_267
Breakpoint 1: where = fib-4.14.1.exe`camlFib__code_begin, address = 0x0000000100003838
(lldb) r
Process 63927 launched: '/Users/tsmc/projects/ocaml/fib-4.14.1.exe' (arm64)
Process 63927 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
    frame #0: 0x0000000100003838 fib-4.14.1.exe`camlFib__code_begin
fib-4.14.1.exe`camlFib__code_begin:
->  0x100003838 <+0>:  sub    sp, sp, #0x20
    0x10000383c <+4>:  str    x30, [sp, #0x18]
    0x100003840 <+8>:  cmp    x0, #0x5
    0x100003844 <+12>: b.ge   0x100003858               ; <+32>
Target 0: (fib-4.14.1.exe) stopped.
(lldb) bt
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
  * frame #0: 0x0000000100003838 fib-4.14.1.exe`camlFib__code_begin
    frame #1: 0x00000001000038ac fib-4.14.1.exe`camlFib__code_begin + 116
    frame #2: 0x0000000100029c44 fib-4.14.1.exe`caml_startup_common(argv=<unavailable>, pooling=<unavailable>) at startup_nat.c:160:9 [opt]
    frame #3: 0x0000000100029cb8 fib-4.14.1.exe`caml_main [inlined] caml_startup_exn(argv=<unavailable>) at startup_nat.c:167:10 [opt]
    frame #4: 0x0000000100029cb0 fib-4.14.1.exe`caml_main [inlined] caml_startup(argv=<unavailable>) at startup_nat.c:172:15 [opt]
    frame #5: 0x0000000100029cb0 fib-4.14.1.exe`caml_main(argv=<unavailable>) at startup_nat.c:179:3 [opt]
    frame #6: 0x0000000100029d18 fib-4.14.1.exe`main(argc=<unavailable>, argv=<unavailable>) at main.c:37:3 [opt]
    frame #7: 0x000000018863d0e0 dyld`start + 2360
```

Observe that I have the full backtrace all the way from the main function in the runtime.

You can keep continuing and the backtrace continues to build up, showing the recursive calls to fib.

``` ocaml
(lldb) bt
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
  * frame #0: 0x0000000100003838 fib-4.14.1.exe`camlFib__code_begin
    frame #1: 0x0000000100003864 fib-4.14.1.exe`camlFib__code_begin + 44
    frame #2: 0x00000001000038ac fib-4.14.1.exe`camlFib__code_begin + 116
    frame #3: 0x0000000100029c44 fib-4.14.1.exe`caml_startup_common(argv=<unavailable>, pooling=<unavailable>) at startup_nat.c:160:9 [opt]
    frame #4: 0x0000000100029cb8 fib-4.14.1.exe`caml_main [inlined] caml_startup_exn(argv=<unavailable>) at startup_nat.c:167:10 [opt]
    frame #5: 0x0000000100029cb0 fib-4.14.1.exe`caml_main [inlined] caml_startup(argv=<unavailable>) at startup_nat.c:172:15 [opt]
    frame #6: 0x0000000100029cb0 fib-4.14.1.exe`caml_main(argv=<unavailable>) at startup_nat.c:179:3 [opt]
    frame #7: 0x0000000100029d18 fib-4.14.1.exe`main(argc=<unavailable>, argv=<unavailable>) at main.c:37:3 [opt]
    frame #8: 0x000000018863d0e0 dyld`start + 2360
(lldb) c
Process 63927 resuming
Process 63927 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
    frame #0: 0x0000000100003838 fib-4.14.1.exe`camlFib__code_begin
fib-4.14.1.exe`camlFib__code_begin:
->  0x100003838 <+0>:  sub    sp, sp, #0x20
    0x10000383c <+4>:  str    x30, [sp, #0x18]
    0x100003840 <+8>:  cmp    x0, #0x5
    0x100003844 <+12>: b.ge   0x100003858               ; <+32>
Target 0: (fib-4.14.1.exe) stopped.
(lldb) bt
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
  * frame #0: 0x0000000100003838 fib-4.14.1.exe`camlFib__code_begin
    frame #1: 0x0000000100003864 fib-4.14.1.exe`camlFib__code_begin + 44
    frame #2: 0x0000000100003864 fib-4.14.1.exe`camlFib__code_begin + 44
    frame #3: 0x00000001000038ac fib-4.14.1.exe`camlFib__code_begin + 116
    frame #4: 0x0000000100029c44 fib-4.14.1.exe`caml_startup_common(argv=<unavailable>, pooling=<unavailable>) at startup_nat.c:160:9 [opt]
    frame #5: 0x0000000100029cb8 fib-4.14.1.exe`caml_main [inlined] caml_startup_exn(argv=<unavailable>) at startup_nat.c:167:10 [opt]
    frame #6: 0x0000000100029cb0 fib-4.14.1.exe`caml_main [inlined] caml_startup(argv=<unavailable>) at startup_nat.c:172:15 [opt]
    frame #7: 0x0000000100029cb0 fib-4.14.1.exe`caml_main(argv=<unavailable>) at startup_nat.c:179:3 [opt]
    frame #8: 0x0000000100029d18 fib-4.14.1.exe`main(argc=<unavailable>, argv=<unavailable>) at main.c:37:3 [opt]
    frame #9: 0x000000018863d0e0 dyld`start + 2360
(lldb)
```

Here we are on MacOS / ARM64, so while the values cannot be printed directly, I know that the arguments are sent in registers, for ARM64 the first 4 arguments are passed in registers x0-x3. I can examine the value at entry to the function like so:

``` ocaml
(lldb) p $x0 >> 1
(unsigned long) 16
(lldb)  
```
right shifting by 1 due to OCaml value representation, which uses 31 bits for integer values.

``` ocaml
(lldb) c
Process 63927 resuming
Process 63927 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
    frame #0: 0x0000000100003838 fib-4.14.1.exe`camlFib__code_begin
fib-4.14.1.exe`camlFib__code_begin:
->  0x100003838 <+0>:  sub    sp, sp, #0x20
    0x10000383c <+4>:  str    x30, [sp, #0x18]
    0x100003840 <+8>:  cmp    x0, #0x5
    0x100003844 <+12>: b.ge   0x100003858               ; <+32>
Target 0: (fib-4.14.1.exe) stopped.
(lldb) p $x0 >> 1
(unsigned long) 14
```

From this point you can run the entire OCaml program, setting breakpoints and interacting with it as you would a regular C/C++ program.

## Issues
* Setting breakpoints using line numbers eg `br s -f fib.ml -l 6` does not work in 4.14 or 5.*
* Setting breakpoints using symbols eg `br s -n camlFib__fib_267` is broken in OCaml 5.1 onwards. OCaml 5.1 changed name mangling to use `.` separators over `__` which breaks lldb on MacOS. Linux LLDB is unaffected.
* Backtraces in 4.14 onwards show offset `camlFib__code_begin + 116` rather than line number in source code
* Backtraces in 5.0 onwards missing C code for OCaml runtime setup, beginning at `camlFib__code_begin`

