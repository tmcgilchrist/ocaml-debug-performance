# Sample GDB on Linux debugging session

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
$ ocamlopt -g -o fib.exe fib.ml
```

Here the OCaml from our fib program gets name mangled into the following:

``` ocaml
$ nm -pa fib.exe|grep "camlFib"
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
$ ocamlopt -dlambda -g -S -o fib.exe fib.ml
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
$ gdb fib.exe
GNU gdb (Ubuntu 14.0.50.20230907-0ubuntu1) 14.0.50.20230907-git
Copyright (C) 2023 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "aarch64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from fib.exe...
(gdb) break camlFib__fib_267
Breakpoint 1 at 0x4f9b8: file fib.ml, line 1.
(gdb) r
Starting program: /home/tsmc/ocaml/fib-4.14.1.exe 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/aarch64-linux-gnu/libthread_db.so.1".

Breakpoint 1, camlFib__fib_267 () at fib.ml:1
1	let rec fib n =
(gdb) bt
#0  camlFib__fib_267 () at fib.ml:1
#1  0x0000aaaaaaaefa2c in camlFib__main_269 () at fib.ml:6
#2  0x0000aaaaaaaefa98 in camlFib__entry () at fib.ml:9
#3  0x0000aaaaaaaec0a4 in caml_program ()
#4  0x0000aaaaaab337d4 in caml_start_program ()
#5  0x0000aaaaaab34090 in caml_startup_common (argv=0xaaaaaab709c8, 
    pooling=<optimized out>, pooling@entry=0) at startup_nat.c:160
#6  0x0000aaaaaab34110 in caml_startup_exn (argv=<optimized out>)
    at startup_nat.c:167
#7  caml_startup (argv=<optimized out>) at startup_nat.c:172
#8  caml_main (argv=<optimized out>) at startup_nat.c:179
#9  0x0000aaaaaaaebdd0 in main (argc=<optimized out>, argv=<optimized out>)
    at main.c:37
(gdb) 

```

Observe that I have the full backtrace all the way from the main function in the runtime.

You can keep continuing and the backtrace continues to build up, showing the recursive calls to fib.

``` ocaml
(gdb) c
Continuing.

Breakpoint 1, camlFib__fib_267 () at fib.ml:1
1	let rec fib n =
(gdb) bt
#0  camlFib__fib_267 () at fib.ml:1
#1  0x0000aaaaaaaef9e4 in camlFib__fib_267 () at fib.ml:3
#2  0x0000aaaaaaaefa2c in camlFib__main_269 () at fib.ml:6
#3  0x0000aaaaaaaefa98 in camlFib__entry () at fib.ml:9
#4  0x0000aaaaaaaec0a4 in caml_program ()
#5  0x0000aaaaaab337d4 in caml_start_program ()
#6  0x0000aaaaaab34090 in caml_startup_common (argv=0xaaaaaab709c8, pooling=<optimized out>, pooling@entry=0)
    at startup_nat.c:160
#7  0x0000aaaaaab34110 in caml_startup_exn (argv=<optimized out>) at startup_nat.c:167
#8  caml_startup (argv=<optimized out>) at startup_nat.c:172
#9  caml_main (argv=<optimized out>) at startup_nat.c:179
#10 0x0000aaaaaaaebdd0 in main (argc=<optimized out>, argv=<optimized out>) at main.c:37
(gdb) c
Continuing.

Breakpoint 1, camlFib__fib_267 () at fib.ml:1
1	let rec fib n =
(gdb) bt
#0  camlFib__fib_267 () at fib.ml:1
#1  0x0000aaaaaaaef9e4 in camlFib__fib_267 () at fib.ml:3
#2  0x0000aaaaaaaef9e4 in camlFib__fib_267 () at fib.ml:3
#3  0x0000aaaaaaaefa2c in camlFib__main_269 () at fib.ml:6
#4  0x0000aaaaaaaefa98 in camlFib__entry () at fib.ml:9
#5  0x0000aaaaaaaec0a4 in caml_program ()
#6  0x0000aaaaaab337d4 in caml_start_program ()
#7  0x0000aaaaaab34090 in caml_startup_common (argv=0xaaaaaab709c8, pooling=<optimized out>, pooling@entry=0)
    at startup_nat.c:160
#8  0x0000aaaaaab34110 in caml_startup_exn (argv=<optimized out>) at startup_nat.c:167
#9  caml_startup (argv=<optimized out>) at startup_nat.c:172
#10 caml_main (argv=<optimized out>) at startup_nat.c:179
#11 0x0000aaaaaaaebdd0 in main (argc=<optimized out>, argv=<optimized out>) at main.c:37
(gdb)
```

Here we are on Linux / ARM64, so while the values cannot be printed directly, I know that the arguments are sent in registers, for ARM64 the first 4 arguments are passed in registers x0-x3. I can examine the value at entry to the function like so:

``` ocaml
(gdb) p $x0 >> 1
$1 = 16
(gdb) 
```
right shifting by 1 due to OCaml value representation, which uses 31 bits for integer values.

``` ocaml
(gdb) c
Continuing.

Breakpoint 1, camlFib__fib_267 () at fib.ml:1

(gdb) p $x0 >> 1
$3 = 14
```

You can also set break points based on the line numbers in gbd.

``` ocaml
(gdb) list
1	let rec fib n =
2	  if n < 2 then 1
3	  else fib (n-1) + fib (n-2)
4	
5	let main () =
6	  let r = fib 20 in
7	  Printf.printf "fib(20) = %d" r
8	
9	let _ = main ()
(gdb) break fib.ml:6
Breakpoint 2 at 0xaaaaaaaefa28: file fib.ml, line 6.
```

From this point you can run the entire OCaml program, setting breakpoints and interacting with it as you would a regular C/C++ program. 

