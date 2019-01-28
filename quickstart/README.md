These instructions lead you through setup and fuzzing of a sample program.

Setup
========

Jump to the appropriate part of this Setup section based on what you're
configuring, then go to the next section (Building AFL).

Logging in to the provided instance
-------------------------------------

If you're reading these instructions then you've probably already made it! Skip to the Building AFL section.

Running the docker image locally
-----------------------------------

See the "Running locally" section of docker/README.md, then skip to the Building AFL section.

Setting up your own machine manually
---------------------------------------

Install dependencies:

    $ sudo apt-get install clang-4.0 build-essential llvm-4.0-dev gnuplot-nox

Work around some Ubuntu annoyances

    $ sudo update-alternatives --install /usr/bin/clang clang `which clang-4.0` 1
    $ sudo update-alternatives --install /usr/bin/clang++ clang++ `which clang++-4.0` 1
    $ sudo update-alternatives --install /usr/bin/llvm-config llvm-config `which llvm-config-4.0` 1
    $ sudo update-alternatives --install /usr/bin/llvm-symbolizer llvm-symbolizer `which llvm-symbolizer-4.0` 1

Make system not interfere with crash detection:

    $ echo core | sudo tee /proc/sys/kernel/core_pattern

Get afl:

    $ cd
    $ wget http://lcamtuf.coredump.cx/afl/releases/afl-latest.tgz
    $ tar xvf afl-latest.tgz

Building AFL
============

    $ cd afl-2.45b   # replace with whatever the current version is
    $ make
    $ make -C llvm_mode


The `vulnerable` program
========================

Build our quickstart program using the instrumented compiler:

    $ cd /path/to/quickstart # (e.g. ~/afl-training/quickstart)
    $ CC=~/afl-2.45b/afl-clang-fast AFL_HARDEN=1 make

Test it:

    $ ./vulnerable
    # Press enter to get usage instructions.
    # Test it on one of the provided inputs:
    $ ./vulnerable < inputs/c


Fuzzing
=======

Fuzz it:

    $ ~/afl-2.45b/afl-fuzz -i inputs -o out ./vulnerable

For comparison you could also test without the provided example inputs, e.g.:

    $ mkdir in
    $ echo "my seed" > in/a
    $ ~/afl-2.45b/afl-fuzz -i in -o out ./vulnerable

POC
====

Without instrumentation (`-n`): (`afl-fuzz -i in -o out -n ./vulnerable`)
![imaing](https://github.com/enovella/afl-training/blob/master/quickstart/pics/afl-pic-vulnerable-dumbf.png)

With instrumentation (`afl-clang-fast`):
![imaing](https://github.com/enovella/afl-training/blob/master/quickstart/pics/afl-pic-vulnerable.png)

Crashes
========

```c
>  ./vulnerable < out/crashes/id\:000000\,sig\:06\,src\:000002\,op\:flip2\,pos\:4
=================================================================
==6075==ERROR: AddressSanitizer: stack-buffer-underflow on address 0x7ffdb77de9a0 at pc 0x00000050be42 bp 0x7ffdb77de990 sp 0x7ffdb77de988
WRITE of size 1 at 0x7ffdb77de9a0 thread T0
    #0 0x50be41  (/home/edu/github/afl-training/quickstart/vulnerable+0x50be41)
    #1 0x7f0ad25aab96  (/lib/x86_64-linux-gnu/libc.so.6+0x21b96)
    #2 0x41c339  (/home/edu/github/afl-training/quickstart/vulnerable+0x41c339)

Address 0x7ffdb77de9a0 is located in stack of thread T0 at offset 0 in frame
    #0 0x50b80f  (/home/edu/github/afl-training/quickstart/vulnerable+0x50b80f)

  This frame has 1 object(s):
    [32, 132) 'input'
HINT: this may be a false positive if your program uses some custom stack unwind mechanism or swapcontext
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: stack-buffer-underflow (/home/edu/github/afl-training/quickstart/vulnerable+0x50be41)
Shadow bytes around the buggy address:
  0x100036ef3ce0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100036ef3cf0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100036ef3d00: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100036ef3d10: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100036ef3d20: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x100036ef3d30: 00 00 00 00[f1]f1 f1 f1 00 00 00 00 00 00 00 00
  0x100036ef3d40: 00 00 00 00 04 f3 f3 f3 f3 f3 f3 f3 00 00 00 00
  0x100036ef3d50: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100036ef3d60: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100036ef3d70: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100036ef3d80: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==6075==ABORTING
```

```
>  ./vulnerable < out/crashes/id\:000001\,sig\:06\,src\:000004\,op\:havoc\,rep\:64
=================================================================
==6113==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7ffe813fc0c4 at pc 0x00000044d7e4 bp 0x7ffe813fc030 sp 0x7ffe813fb7e0
READ of size 101 at 0x7ffe813fc0c4 thread T0
    #0 0x44d7e3  (/home/edu/github/afl-training/quickstart/vulnerable+0x44d7e3)
    #1 0x50bc47  (/home/edu/github/afl-training/quickstart/vulnerable+0x50bc47)
    #2 0x7f59254fdb96  (/lib/x86_64-linux-gnu/libc.so.6+0x21b96)
    #3 0x41c339  (/home/edu/github/afl-training/quickstart/vulnerable+0x41c339)

Address 0x7ffe813fc0c4 is located in stack of thread T0 at offset 132 in frame
    #0 0x50b80f  (/home/edu/github/afl-training/quickstart/vulnerable+0x50b80f)

  This frame has 1 object(s):
    [32, 132) 'input' <== Memory access at offset 132 overflows this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism or swapcontext
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: stack-buffer-overflow (/home/edu/github/afl-training/quickstart/vulnerable+0x44d7e3)
Shadow bytes around the buggy address:
  0x1000502777c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1000502777d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1000502777e0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1000502777f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100050277800: 00 00 00 00 00 00 00 00 f1 f1 f1 f1 00 00 00 00
=>0x100050277810: 00 00 00 00 00 00 00 00[04]f3 f3 f3 f3 f3 f3 f3
  0x100050277820: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100050277830: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100050277840: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100050277850: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100050277860: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==6113==ABORTING
```