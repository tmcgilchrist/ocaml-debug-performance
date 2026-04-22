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
* Setting breakpoints using line numbers eg `br s -f fib.ml -l 6` does not work in 4.14, 5.* or flambda2 (ARM64).
* Setting breakpoints using symbols eg `br s -n camlFib__fib_267` is broken in OCaml 5.1 onwards. OCaml 5.1 changed name mangling to use `.` separators over `__` which breaks lldb on MacOS. Linux LLDB is unaffected.
* Backtraces in 4.14 onwards show offset `camlFib__code_begin + 116` rather than line number in source code
* Backtraces in 5.0 onwards missing C code for OCaml runtime setup, beginning at `camlFib__code_begin`

## Wed  6 Mar 14:48:12 2024

``` shell
$ cat fib.ml
let rec fib n =
  if n < 2 then 1
  else fib (n-1) + fib (n-2)

let main () =
  let r = fib 20 in
  Printf.printf "fib(20) = %d" r

let _ = main ()

$ ocamlopt -g -o fib.exe fib.ml

$ dsymutil --symtab fib.exe |grep camlFib
----------------------------------------------------------------------
Symbol table for: 'fib.exe' (arm64)
----------------------------------------------------------------------
Index    n_strx   n_type             n_sect n_desc n_value
======== -------- ------------------ ------ ------ ----------------
[     0] 00013fc5 0e (     SECT    ) 01     0000   0000000100035590 '_caml_array_gather'
....
[  5471] 000057e7 0f (     SECT EXT) 09     0200   000000010006da60 '_camlFib'
[  5472] 000057f0 0f (     SECT EXT) 09     0200   000000010006daf8 '_camlFib$1'
[  5473] 000057fb 0f (     SECT EXT) 09     0200   000000010006dad0 '_camlFib$2'
[  5474] 00005806 0f (     SECT EXT) 09     0200   000000010006dab8 '_camlFib$3'
[  5475] 00005811 0f (     SECT EXT) 09     0200   000000010006daa0 '_camlFib$4'
[  5476] 0000581c 0f (     SECT EXT) 09     0200   000000010006da88 '_camlFib$5'
[  5477] 00005827 0f (     SECT EXT) 09     0200   000000010006da48 '_camlFib$6'
[  5478] 00005832 0f (     SECT EXT) 09     0200   000000010006da30 '_camlFib$7'
[  5479] 0000583d 0f (     SECT EXT) 01     0000   00000001000071f8 '_camlFib$code_begin'
[  5480] 00005851 0f (     SECT EXT) 01     0200   0000000100007394 '_camlFib$code_end'
[  5481] 00005863 0f (     SECT EXT) 09     0000   000000010006da28 '_camlFib$data_begin'
[  5482] 00005877 0f (     SECT EXT) 09     0200   000000010006db10 '_camlFib$data_end'
[  5483] 00005889 0f (     SECT EXT) 01     0200   0000000100007318 '_camlFib$entry'
[  5484] 00005898 0f (     SECT EXT) 01     0200   0000000100007210 '_camlFib$fib_270'
[  5485] 000058a9 0f (     SECT EXT) 09     0200   000000010006db18 '_camlFib$frametable'
[  5486] 000058bd 0f (     SECT EXT) 09     0200   000000010006da70 '_camlFib$gc_roots'
[  5487] 000058cf 0f (     SECT EXT) 01     0200   00000001000072a0 '_camlFib$main_272'

$ nm -pa fib.exe |grep camlFib
000000010006da60 D _camlFib
000000010006daf8 D _camlFib$1
000000010006dad0 D _camlFib$2
000000010006dab8 D _camlFib$3
000000010006daa0 D _camlFib$4
000000010006da88 D _camlFib$5
000000010006da48 D _camlFib$6
000000010006da30 D _camlFib$7
00000001000071f8 T _camlFib$code_begin
0000000100007394 T _camlFib$code_end
000000010006da28 D _camlFib$data_begin
000000010006db10 D _camlFib$data_end
0000000100007318 T _camlFib$entry
0000000100007210 T _camlFib$fib_270
000000010006db18 D _camlFib$frametable
000000010006da70 D _camlFib$gc_roots
00000001000072a0 T _camlFib$main_272
```

``` ocaml
image lookup -r -n camlFib
5 matches found in /Users/tsmc/code/ocaml/ocaml/fib-5.3.0.exe:
        Address: fib-5.3.0.exe[0x00000001000071f8] (fib-5.3.0.exe.__TEXT.__text + 11408)
        Summary: fib-5.3.0.exe`camlFib$code_begin        Address: fib-5.3.0.exe[0x0000000100007394] (fib-5.3.0.exe.__TEXT.__text + 11820)
        Summary: fib-5.3.0.exe`camlFib$code_begin + 412        Address: fib-5.3.0.exe[0x0000000100007318] (fib-5.3.0.exe.__TEXT.__text + 11696)
        Summary: fib-5.3.0.exe`camlFib$code_begin + 288        Address: fib-5.3.0.exe[0x0000000100007210] (fib-5.3.0.exe.__TEXT.__text + 11432)
        Summary: fib-5.3.0.exe`camlFib$code_begin + 24        Address: fib-5.3.0.exe[0x00000001000072a0] (fib-5.3.0.exe.__TEXT.__text + 11576)
        Summary: fib-5.3.0.exe`camlFib$code_begin + 168
```

Noticing that the OCaml executable has entries for OCaml compiler runtime but not the `fib` executable.
We need OSO debug information like the Rust example above.
See `compile_unit_proto_die` from `flambda-backend` repo that apparently emits the right SO/OSO combinations.
Need to validate this works. Off to build flambda-backend on MacOS.

Hacked assembly file to include `.file fib.ml` at the top of the file.

Using flambda we get these debug symbols:

``` ocaml
(lldb) image lookup -r -n camlFib
5 matches found in /Users/tsmc/code/ocaml/flambda-backend/fib-jst.exe:
        Address: fib-jst.exe[0x0000000100003518] (fib-jst.exe.__TEXT.__text + 10448)
        Summary: fib-jst.exe`camlFib__code_begin        Address: fib-jst.exe[0x00000001000035e0] (fib-jst.exe.__TEXT.__text + 10648)
        Summary: fib-jst.exe`camlCamlinternalFormatBasics__code_begin        Address: fib-jst.exe[0x00000001000035c0] (fib-jst.exe.__TEXT.__text + 10616)
        Summary: fib-jst.exe`camlFib__entry        Address: fib-jst.exe[0x0000000100003520] (fib-jst.exe.__TEXT.__text + 10456)
        Summary: fib-jst.exe`camlFib__fib_0_2_code        Address: fib-jst.exe[0x0000000100003578] (fib-jst.exe.__TEXT.__text + 10544)
        Summary: fib-jst.exe`camlFib__main_1_3_code
```
and the backtrace has C and OCaml frames:
``` ocaml
(lldb) bt
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 11.1
  * frame #0: 0x0000000100003520 fib-jst.exe`camlFib__fib_0_2_code
    frame #1: 0x000000010000354c fib-jst.exe`camlFib__fib_0_2_code + 44
    frame #2: 0x000000010000354c fib-jst.exe`camlFib__fib_0_2_code + 44
    frame #3: 0x000000010000354c fib-jst.exe`camlFib__fib_0_2_code + 44
    frame #4: 0x0000000100003588 fib-jst.exe`camlFib__main_1_3_code + 16
    frame #5: 0x00000001000035d0 fib-jst.exe`camlFib__entry + 16
    frame #6: 0x0000000100000c84 fib-jst.exe`caml_program + 52
    frame #7: 0x000000010005718c fib-jst.exe`caml_start_program + 104
    frame #8: 0x00000001000309f0 fib-jst.exe`caml_startup_common(argv=0x000000016fdfedc8, pooling=<unavailable>) at startup_nat.c:165:9 [opt]
    frame #9: 0x0000000100030a6c fib-jst.exe`caml_main [inlined] caml_startup_exn(argv=<unavailable>) at startup_nat.c:175:10 [opt]
    frame #10: 0x0000000100030a64 fib-jst.exe`caml_main [inlined] caml_startup(argv=<unavailable>) at startup_nat.c:180:15 [opt]
    frame #11: 0x0000000100030a64 fib-jst.exe`caml_main(argv=<unavailable>) at startup_nat.c:187:3 [opt]
    frame #12: 0x0000000100030acc fib-jst.exe`main(argc=<unavailable>, argv=<unavailable>) at main.c:37:3 [opt]
    frame #13: 0x0000000186a9d0e0 dyld`start + 2360

```

OCaml 5.3 +trunk

``` ocaml
(lldb) image lookup -r -n camlFib
5 matches found in /Users/tsmc/code/ocaml/ocaml/fib-5.3.0.exe:
        Address: fib-5.3.0.exe[0x00000001000071f8] (fib-5.3.0.exe.__TEXT.__text + 11408)
        Summary: fib-5.3.0.exe`camlFib$code_begin        Address: fib-5.3.0.exe[0x0000000100007394] (fib-5.3.0.exe.__TEXT.__text + 11820)
        Summary: fib-5.3.0.exe`camlFib$code_begin + 412        Address: fib-5.3.0.exe[0x0000000100007318] (fib-5.3.0.exe.__TEXT.__text + 11696)
        Summary: fib-5.3.0.exe`camlFib$code_begin + 288        Address: fib-5.3.0.exe[0x0000000100007210] (fib-5.3.0.exe.__TEXT.__text + 11432)
        Summary: fib-5.3.0.exe`camlFib$code_begin + 24        Address: fib-5.3.0.exe[0x00000001000072a0] (fib-5.3.0.exe.__TEXT.__text + 11576)
        Summary: fib-5.3.0.exe`camlFib$code_begin + 168

(lldb) br s -n camlFib$main_272
Breakpoint 1: where = fib-5.3.0.exe`camlFib$code_begin + 168, address = 0x00000001000072a0
```

In the backtrace we have this for C runtime files:

``` ocaml 
0000000000000000 - 01 0000    SO 
0000000000000000 - 00 0000    SO /Users/tsmc/code/ocaml/flambda-backend/_build/runtime_stdlib/ocaml/runtime4/
0000000000000000 - 00 0000    SO codefrag.c
0000000065e99e6e - 00 0001   OSO /Users/tsmc/code/ocaml/flambda-backend/lib/ocaml/libasmrun.a(codefrag.n.o)
0000000100056d58 - 01 0000 BNSYM 
0000000100056d58 - 01 0000   FUN _caml_register_code_fragment
00000000000000bc - 00 0000   FUN 
0000000100056d58 - 01 0000 ENSYM 
...
000000010008afa0 S _Caml_state
0000000100000000 T __mh_execute_header
0000000100078eb0 D _camlCamlinternalFormat
0000000100069c30 D _camlCamlinternalFormatBasics
```

Build using verbose commands, dump lambda representation and assembly file:
``` ocaml
$ opam exec --switch="4.14.1" -- ocamlopt -dlambda -verbose -g -S -o fib-4.14.1.exe fib.ml 
(seq
  (letrec
    (fib/267
       (function n/268[int] : int
         (if (== n/268 0) 0
           (if (== n/268 1) 1
             (+ (apply fib/267 (- n/268 1)) (apply fib/267 (- n/268 2)))))))
    (setfield_ptr(root-init) 0 (global Fib!) fib/267))
  (let
    (main/269 =
       (function param/308[int] : int
         (let (r/271 =[int] (apply (field 0 (global Fib!)) 200))
           (apply (field 1 (global Stdlib__Printf!))
             [0: [11: "fib(200) = " [4: 0 0 0 0]] "fib(200) = %d"] r/271))))
    (setfield_ptr(root-init) 1 (global Fib!) main/269))
  (apply (field 1 (global Fib!)) 0) 0 0)
+ cc -c -Wno-trigraphs  -o 'fib.o' 'fib.s'
+ cc -c -Wno-trigraphs  -o '/var/folders/z_/7yzlrkjn6pd441zs1qhzpjv00000gn/T/camlstartupc4c0a3.o' '/var/folders/z_/7yzlrkjn6pd441zs1qhzpjv00000gn/T/camlstartup0f5911.s'
+ cc -O2 -fno-strict-aliasing -fwrapv -pthread -Wall -Wdeclaration-after-statement -fno-common    -Wl,-no_compact_unwind -o 'fib-4.14.1.exe'  '-L/Users/tsmc/.opam/4.14.1/lib/ocaml'  '/var/folders/z_/7yzlrkjn6pd441zs1qhzpjv00000gn/T/camlstartupc4c0a3.o' '/Users/tsmc/.opam/4.14.1/lib/ocaml/std_exit.o' 'fib.o' '/Users/tsmc/.opam/4.14.1/lib/ocaml/stdlib.a' '/Users/tsmc/.opam/4.14.1/lib/ocaml/libasmrun.a' -lm 

```

Looking at cargo/rust binaries compiled with Debug turned on

``` ocaml
$ nm -pa target/debug/dwarfdump
....
0000000000000000 - 01 0000    SO 
0000000000000000 - 00 0000    SO /Users/tsmc/code/rust/gimli/crates/examples/src/bin/dwarfdump.rs/@/
0000000000000000 - 00 0000    SO 16r479fmyrfz78wy
0000000000000000 - 00 0001   OSO /Users/tsmc/code/rust/gimli/target/debug/deps/dwarfdump-7bf704beafb18da7.16r479fmyrfz78wy.rcgu.o
0000000100006850 - 01 0000 BNSYM 
0000000100006850 - 01 0000   FUN __ZN42_$LT$$RF$T$u20$as$u20$core..fmt..Debug$GT$3fmt17hd091cb1a61463214E
0000000000000034 - 00 0000   FUN 
0000000100006850 - 01 0000 ENSYM 
```

We get a section of
SO 
SO <path to source file>
SO <hased value>
OSO <object file>
BNSYM
FUN <some symbol in source file>
FUN
ENSYM

Adding `.file 0 "/home/tsmc" "fib.c"` section to the top of the assembly generated by each compilation unit. 
Restores an accurate backtrace and improves setting breakpoints based on symbol names.

``` ocaml
(lldb) bt
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 2.1
  * frame #0: 0x0000000100004f98 fib-test.exe`camlFib$fib_270
    frame #1: 0x0000000100004fec fib-test.exe`camlFib$fib_270 + 84
    frame #2: 0x0000000100005054 fib-test.exe`camlFib$main_272 + 44
    frame #3: 0x000000010000510c fib-test.exe`camlFib$entry + 108
    frame #4: 0x00000001000024e4 fib-test.exe`caml_program + 476
    frame #5: 0x0000000100060124 fib-test.exe`caml_start_program + 132
    frame #6: 0x00000001000024e4 fib-test.exe`caml_program + 476
(lldb) image lookup -r -n camlFib
5 matches found in /Users/tsmc/code/ocaml/ocaml/fib-test.exe:
        Address: fib-test.exe[0x0000000100004f80] (fib-test.exe.__TEXT.__text + 11408)
        Summary: fib-test.exe`camlFib$code_begin        Address: fib-test.exe[0x000000010000511c] (fib-test.exe.__TEXT.__text + 11820)
        Summary: fib-test.exe`camlFib$code_end        Address: fib-test.exe[0x00000001000050a0] (fib-test.exe.__TEXT.__text + 11696)
        Summary: fib-test.exe`camlFib$entry        Address: fib-test.exe[0x0000000100004f98] (fib-test.exe.__TEXT.__text + 11432)
        Summary: fib-test.exe`camlFib$fib_270        Address: fib-test.exe[0x0000000100005028] (fib-test.exe.__TEXT.__text + 11576)
        Summary: fib-test.exe`camlFib$main_272
(lldb) br list
Current breakpoints:
1: name = 'camlFib$main_272', locations = 1, resolved = 1, hit count = 1
  1.1: where = fib-test.exe`camlFib$main_272, address = 0x0000000100005028, resolved, hit count = 1 

2: name = 'camlFib$fib_270', locations = 1, resolved = 1, hit count = 2
  2.1: where = fib-test.exe`camlFib$fib_270, address = 0x0000000100004f98, resolved, hit count = 2 

```

LLVM doesn't allow STABS directives in assembly files but seems to generate some of the information using
`.file` and `.loc` directives. OCaml doesn't generate `.loc` equivalent to clang. It needs to be modified to use
the more verbose `	.loc	1 4 14 is_stmt 0                ; fib.c:4:14` sections. The CFI generated by ARM64
might not be correct, seems like we don't generate enough details.

LOC showing `.stab` directives being rejected https://github.com/llvm/llvm-project/blob/release/15.x/llvm/lib/MC/MCParser/AsmParser.cpp#L3729. In theory we could override using llvm-as and use GNU as instead, this would yield direct STABS support. Probably should use the provided tooling.

The "stabs" debug format - https://opensource.apple.com/source/gdb/gdb-250/doc/stabs.pdf

Compared to Flambda2 JST we are missing C parts of the stack trace and locations of the source code

``` ocaml
    frame #7: 0x000000010005718c fib-jst.exe`caml_start_program + 104
    frame #8: 0x00000001000309f0 fib-jst.exe`caml_startup_common(argv=0x000000016fdfedc8, pooling=<unavailable>) at startup_nat.c:165:9 [opt]
    frame #9: 0x0000000100030a6c fib-jst.exe`caml_main [inlined] caml_startup_exn(argv=<unavailable>) at startup_nat.c:175:10 [opt]
    frame #10: 0x0000000100030a64 fib-jst.exe`caml_main [inlined] caml_startup(argv=<unavailable>) at startup_nat.c:180:15 [opt]
```


## Mon 18 Mar 10:47:51 2024

 Flambda / 4.14 -> Missing source mappings to ML files
 5.0.0 -> truncated stack without C frames
 5.1.1 -> Can't set named breakpoints
       -> truncated stack without C frames
 5.3   -> name mangling restores setting named breakpoints
       -> Still contains truncated stack without C frames.

Why? Missing framepointers? Especially in jump to caml_start_program

## Mon  1 Apr 10:28:28 2024

Adding debugger support for your target https://llvm.org/devmtg/2016-03/Tutorials/LLDB-tutorial.pdf
Useful for debugging why symbols don't load.

Use `log enable lldb unwind`

``` shell
fib-test-2.exe`camlCamlinternalFormatBasics$entry:
->  0x100006f28 <+32>: ldr    x16, [x28, #0x40]
    0x100006f2c <+36>: mov    sp, x16
    0x100006f30 <+40>: bl     0x100064440               ; symbol stub for: caml_system__code_end + 168
    0x100006f34 <+44>: mov    sp, x29
Target 0: (fib-test-2.exe) stopped.
(lldb) bt
 th1/fr0 supplying caller's saved fp (29)'s location, cached
 th1/fr0 requested caller's saved PC but this UnwindPlan uses a RA reg; getting lr (30) instead
 th1/fr0 supplying caller's saved lr (30)'s location using eh_frame CFI UnwindPlan
 th1/fr0 supplying caller's register lr (30) from the stack, saved at CFA plus offset -8 [saved at 0x148030008]
 th1/fr0 requested caller's saved PC but this UnwindPlan uses a RA reg; getting lr (30) instead
 th1/fr0 supplying caller's saved lr (30)'s location using eh_frame CFI UnwindPlan
 th1/fr0 supplying caller's register lr (30) from the stack, saved at CFA plus offset -8 [saved at 0x148030008]
  th1/fr1 pc = 0x100002fdc
 th1/fr0 supplying caller's saved fp (29)'s location, cached
  th1/fr1 fp = 0x148030000
 th1/fr0 supplying caller's saved sp (31)'s location, cached
  th1/fr1 sp = 0x148030010
  th1/fr1 with pc value of 0x100002fdc, symbol name is 'caml_program'
  th1/fr1 Backing up the pc value of 0x100002fdc by 1 and re-doing symbol lookup; old symbol was caml_program
  th1/fr1 Symbol is now caml_program
  th1/fr1 Using full unwind plan 'eh_frame CFI'
  th1/fr1 active row: 0x0000000100002fd4: CFA=sp+16 => lr=[CFA-8] 
 th1/fr0 supplying caller's saved sp (31)'s location, cached
  th1/fr1 CFA is 0x148030020: Register sp (31) contents are 0x148030010, offset is 16
  th1/fr1 m_cfa = 0x148030020 m_afa = 0xffffffffffffffff
  th1/fr1 initialized frame current pc is 0x100002fdb cfa is 0x148030020 afa is 0xffffffffffffffff
 th1/fr0 requested caller's saved PC but this UnwindPlan uses a RA reg; getting lr (30) instead
 th1/fr0 supplying caller's saved lr (30)'s location using eh_frame CFI UnwindPlan
 th1/fr0 supplying caller's register lr (30) from the stack, saved at CFA plus offset -8 [saved at 0x148030008]
  th1/fr1 no save location for fp (29) via 'eh_frame CFI'
 th1/fr0 supplying caller's saved fp (29)'s location, cached
  th1/fr1 requested caller's saved PC but this UnwindPlan uses a RA reg; getting lr (30) instead
  th1/fr1 supplying caller's saved lr (30)'s location using eh_frame CFI UnwindPlan
  th1/fr1 supplying caller's register lr (30) from the stack, saved at CFA plus offset -8 [saved at 0x148030018]
  th1/fr1 requested caller's saved PC but this UnwindPlan uses a RA reg; getting lr (30) instead
  th1/fr1 supplying caller's saved lr (30)'s location using eh_frame CFI UnwindPlan
  th1/fr1 supplying caller's register lr (30) from the stack, saved at CFA plus offset -8 [saved at 0x148030018]
   th1/fr2 pc = 0x1000640b4
  th1/fr1 no save location for fp (29) via 'eh_frame CFI'
 th1/fr0 supplying caller's saved fp (29)'s location, cached
   th1/fr2 fp = 0x148030000
  th1/fr1 supplying caller's saved sp (31)'s location using ABI default
  th1/fr1 supplying caller's register sp (31), value is CFA plus offset 0 [value is 0x148030020]
   th1/fr2 sp = 0x148030020
   th1/fr2 with pc value of 0x1000640b4, symbol name is 'caml_start_program'
   th1/fr2 Backing up the pc value of 0x1000640b4 by 1 and re-doing symbol lookup; old symbol was caml_start_program
   th1/fr2 Symbol is now caml_start_program
   th1/fr2 Using full unwind plan 'EmulateInstructionARM64'
   th1/fr2 active row: 0x0000000100064078: CFA=fp+160 => x8=[CFA-176] x19=[CFA-144] x20=[CFA-136] x21=[CFA-128] x22=[CFA-120] x23=[CFA-112] x24=[CFA-104] x25=[CFA-96] x26=[CFA-88] x27=[CFA-80] x28=[CFA-72] fp=[CFA-160] lr=[CFA-152] d6=[CFA-64] d7=[CFA-56] d8=[CFA-48] d9=[CFA-40] d10=[CFA-32] d11=[CFA-24] d12=[CFA-16] d13=[CFA-8] 
  th1/fr1 no save location for fp (29) via 'eh_frame CFI'
 th1/fr0 supplying caller's saved fp (29)'s location, cached
   th1/fr2 CFA is 0x1480300a0: Register fp (29) contents are 0x148030000, offset is 160
   th1/fr2 m_cfa = 0x1480300a0 m_afa = 0xffffffffffffffff
   th1/fr2 initialized frame current pc is 0x1000640b3 cfa is 0x1480300a0 afa is 0xffffffffffffffff
  th1/fr1 requested caller's saved PC but this UnwindPlan uses a RA reg; getting lr (30) instead
  th1/fr1 supplying caller's saved lr (30)'s location using eh_frame CFI UnwindPlan
  th1/fr1 supplying caller's register lr (30) from the stack, saved at CFA plus offset -8 [saved at 0x148030018]
   th1/fr2 supplying caller's saved fp (29)'s location using EmulateInstructionARM64 UnwindPlan
   th1/fr2 supplying caller's register fp (29) from the stack, saved at CFA plus offset -160 [saved at 0x148030000]
   th1/fr2 requested caller's saved PC but this UnwindPlan uses a RA reg; getting lr (30) instead
   th1/fr2 supplying caller's saved lr (30)'s location using EmulateInstructionARM64 UnwindPlan
   th1/fr2 supplying caller's register lr (30) from the stack, saved at CFA plus offset -152 [saved at 0x148030008]
   th1/fr2 requested caller's saved PC but this UnwindPlan uses a RA reg; getting lr (30) instead
   th1/fr2 supplying caller's saved lr (30)'s location using EmulateInstructionARM64 UnwindPlan
   th1/fr2 supplying caller's register lr (30) from the stack, saved at CFA plus offset -152 [saved at 0x148030008]
    th1/fr3 pc = 0x100002fdc
   th1/fr2 supplying caller's saved fp (29)'s location, cached
    th1/fr3 fp = 0x0
   th1/fr2 supplying caller's saved sp (31)'s location using ABI default
   th1/fr2 supplying caller's register sp (31), value is CFA plus offset 0 [value is 0x1480300a0]
    th1/fr3 sp = 0x1480300a0
    th1/fr3 with pc value of 0x100002fdc, symbol name is 'caml_program'
    th1/fr3 Backing up the pc value of 0x100002fdc by 1 and re-doing symbol lookup; old symbol was caml_program
    th1/fr3 Symbol is now caml_program
    th1/fr3 Using full unwind plan 'eh_frame CFI'
    th1/fr3 active row: 0x0000000100002fd4: CFA=sp+16 => lr=[CFA-8] 
   th1/fr2 supplying caller's saved sp (31)'s location, cached
    th1/fr3 CFA is 0x1480300b0: Register sp (31) contents are 0x1480300a0, offset is 16
    th1/fr3 m_cfa = 0x1480300b0 m_afa = 0xffffffffffffffff
    th1/fr3 initialized frame current pc is 0x100002fdb cfa is 0x1480300b0 afa is 0xffffffffffffffff
   th1/fr2 requested caller's saved PC but this UnwindPlan uses a RA reg; getting lr (30) instead
   th1/fr2 supplying caller's saved lr (30)'s location using EmulateInstructionARM64 UnwindPlan
   th1/fr2 supplying caller's register lr (30) from the stack, saved at CFA plus offset -152 [saved at 0x148030008]
    th1/fr3 no save location for fp (29) via 'eh_frame CFI'
   th1/fr2 supplying caller's saved fp (29)'s location, cached
    th1/fr3 requested caller's saved PC but this UnwindPlan uses a RA reg; getting lr (30) instead
    th1/fr3 supplying caller's saved lr (30)'s location using eh_frame CFI UnwindPlan
    th1/fr3 supplying caller's register lr (30) from the stack, saved at CFA plus offset -8 [saved at 0x1480300a8]
    th1/fr3 requested caller's saved PC but this UnwindPlan uses a RA reg; getting lr (30) instead
    th1/fr3 supplying caller's saved lr (30)'s location using eh_frame CFI UnwindPlan
    th1/fr3 supplying caller's register lr (30) from the stack, saved at CFA plus offset -8 [saved at 0x1480300a8]
     th1/fr4 pc = 0x0
    th1/fr3 no save location for fp (29) via 'eh_frame CFI'
   th1/fr2 supplying caller's saved fp (29)'s location, cached
     th1/fr4 fp = 0x0
    th1/fr3 supplying caller's saved sp (31)'s location using ABI default
    th1/fr3 supplying caller's register sp (31), value is CFA plus offset 0 [value is 0x1480300b0]
     th1/fr4 sp = 0x1480300b0
     th1/fr4 this frame has a pc of 0x0
     Frame 4 invalid RegisterContext for this frame, stopping stack walk
   th1/fr2 requested caller's saved PC but this UnwindPlan uses a RA reg; getting lr (30) instead
   th1/fr2 supplying caller's saved lr (30)'s location using EmulateInstructionARM64 UnwindPlan
   th1/fr2 supplying caller's register lr (30) from the stack, saved at CFA plus offset -152 [saved at 0x148030008]
  th1/fr1 no save location for fp (29) via 'eh_frame CFI'
 th1/fr0 supplying caller's saved fp (29)'s location, cached
   th1/fr2 CFA is 0x148030010: Register fp (29) contents are 0x148030000, offset is 16
   th1/fr2 supplying caller's saved pc (32)'s location using arm64-apple-darwin default unwind plan UnwindPlan
   th1/fr2 supplying caller's register pc (32) from the stack, saved at CFA plus offset -8 [saved at 0x148030008]
   th1/fr2 trying to unwind from this function with the UnwindPlan 'arm64-apple-darwin default unwind plan' because UnwindPlan 'EmulateInstructionARM64' failed.
   th1/fr2 supplying caller's saved fp (29)'s location using arm64-apple-darwin default unwind plan UnwindPlan
   th1/fr2 supplying caller's register fp (29) from the stack, saved at CFA plus offset -16 [saved at 0x148030000]
   th1/fr2 supplying caller's saved pc (32)'s location, cached
   th1/fr2 supplying caller's saved pc (32)'s location, cached
    th1/fr3 pc = 0x100002fdc
   th1/fr2 supplying caller's saved fp (29)'s location, cached
    th1/fr3 fp = 0x0
   th1/fr2 supplying caller's saved sp (31)'s location using arm64-apple-darwin default unwind plan UnwindPlan
   th1/fr2 did not supply reg location for sp (31) because it is volatile
    th1/fr3 with pc value of 0x100002fdc, symbol name is 'caml_program'
    th1/fr3 Backing up the pc value of 0x100002fdc by 1 and re-doing symbol lookup; old symbol was caml_program
    th1/fr3 Symbol is now caml_program
    th1/fr3 Using full unwind plan 'eh_frame CFI'
    th1/fr3 active row: 0x0000000100002fd4: CFA=sp+16 => lr=[CFA-8] 
   th1/fr2 supplying caller's saved sp (31)'s location using arm64-apple-darwin default unwind plan UnwindPlan
   th1/fr2 did not supply reg location for sp (31) because it is volatile
    th1/fr3 failed to get cfa
    Frame 3 invalid RegisterContext for this frame, stopping stack walk
    th1/fr3 no save location for fp (29) via 'eh_frame CFI'
   th1/fr2 supplying caller's saved fp (29)'s location, cached
    th1/fr3 requested caller's saved PC but this UnwindPlan uses a RA reg; getting lr (30) instead
    th1/fr3 supplying caller's saved lr (30)'s location using eh_frame CFI UnwindPlan
    th1/fr3 supplying caller's register lr (30) from the stack, saved at CFA plus offset -8 [saved at 0x1480300a8]
    th1/fr3 requested caller's saved PC but this UnwindPlan uses a RA reg; getting lr (30) instead
    th1/fr3 supplying caller's saved lr (30)'s location using eh_frame CFI UnwindPlan
    th1/fr3 supplying caller's register lr (30) from the stack, saved at CFA plus offset -8 [saved at 0x1480300a8]
     th1/fr4 pc = 0x0
    th1/fr3 no save location for fp (29) via 'eh_frame CFI'
   th1/fr2 supplying caller's saved fp (29)'s location, cached
     th1/fr4 fp = 0x0
    th1/fr3 supplying caller's saved sp (31)'s location, cached
     th1/fr4 sp = 0x1480300b0
     th1/fr4 this frame has a pc of 0x0
     Frame 4 invalid RegisterContext for this frame, stopping stack walk
 th1 Unwind of this thread is complete.
* thread #1, queue = 'com.apple.main-thread', stop reason = instruction step over
  * frame #0: 0x0000000100006f28 fib-test-2.exe`camlCamlinternalFormatBasics$entry + 32
    frame #1: 0x0000000100002fdc fib-test-2.exe`caml_program + 28
    frame #2: 0x00000001000640b4 fib-test-2.exe`caml_start_program + 132
    frame #3: 0x0000000100002fdc fib-test-2.exe`caml_program + 28

```

Tue 21 May 10:12:44 2024
----------
There are two fundamental documents that are useful here:
1. An Apple Library Primer https://forums.developer.apple.com/forums/thread/715385
   This covers the terminology used by Apple developer tools, what tools are used to inspect MachO binaries and
   generally how linking works.

> Apple platforms use DWARF. When you compile a file, the compiler puts the debug info into the resulting object file. When you link a set of object files into a executable, dynamic library, or bundle for distribution, the linker does not include this debug info. Rather, debug info is stored in a separate debug symbols document package. This has the extension .dSYM and is created using dsymutil. Use symbols to learn about the symbols in a file. Use dwarfdump to get detailed information about DWARF debug info. Use atos to map an address to its corresponding symbol name.

2. Apple Lazy DWARF Scheme https://wiki.dwarfstd.org/Apple%27s_%22Lazy%22_DWARF_Scheme.md older document describing the use of STABS information to support DWARF. 


How do we get OSO into object files?

``` shell
 $ ./bin/ocamlopt -verbose -S -g -o fib.exe fib.ml
+ gcc -c -Wno-trigraphs  -o 'fib.o' 'fib.s'
+ gcc -c -Wno-trigraphs  -o '/var/folders/z_/7yzlrkjn6pd441zs1qhzpjv00000gn/T/camlstartup736830.o' '/var/folders/z_/7yzlrkjn6pd441zs1qhzpjv00000gn/T/camlstartup51ae7e.s'
+ gcc -O2 -fno-strict-aliasing -fwrapv -pthread  -pthread   -o 'fib.exe'  '-L/Users/tsmc/projects/ocaml/lib/ocaml'  '/var/folders/z_/7yzlrkjn6pd441zs1qhzpjv00000gn/T/camlstartup736830.o' '/Users/tsmc/projects/ocaml/lib/ocaml/std_exit.o' 'fib.o' '/Users/tsmc/projects/ocaml/lib/ocaml/stdlib.a' '/Users/tsmc/projects/ocaml/lib/ocaml/libasmrun.a'     -lpthread
```

(lldb) image dump symtab fib.exe
Dumps what lldb knows about an executable including these entries for C code

[ 1273]   4701 D   SourceFile      0x0000000000000000                    Sibling -> [ 1284] 0x00640000 /Users/tsmc/projects/ocaml/runtime/dynlink_nat.c
[ 1274]   4703 D   ObjectFile      0x00000000664bfe6b                    0x0000000000000000 0x00660001 /Users/tsmc/code/ocaml/ocaml/lib/ocaml/libasmrun.a(dynlink_nat.n.o)


We've tried using `.file ` directive with DWARF 5 format as per simple compiled C programs which have `.file	0 "/Users/tsmc/projects/ocaml" "prog.c" md5 0xaf28ba27a1bfcef40a3244a7c73faa94` to show the directory plus the file and a checksum. Uses DWARF 5 or
`.file	1 "/Users/tsmc/projects/ocaml" "prog.c"` using DWARF4.

The C parts of the runtime have OSO entries which is good. The object file produced from `fib.s` to `fib.o` doesn't have OSO information included.

`./bin/ocamlopt -dstartup -verbose -S -g -o fib.exe fib.ml` will output the startup assembly file for OCaml.
eg the part that includes caml_program
