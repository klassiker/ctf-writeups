# Generic Flag Checker 1

## Task

Flag Checker Industries™ has released their new product, the Generic Flag Checker®! Aimed at being small, this hand-assembled executable checks your flag in only 8.5kB! Grab yours today!

File: gfc1

## Solution

Before we get started, let's look for strings:

```bash
$ file gfc1
gfc1: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, BuildID[sha1]=7d4bbad2b6eeb736abec4fd52079781dcc333781, stripped
$ rabin2 -z gfc1
[Strings]
nth paddr      vaddr      len size section type  string
―――――――――――――――――――――――――――――――――――――――――――――――――――――――
0   0x00002000 0x00402000 49  49   .rodata ascii nactf{un10ck_th3_s3cr3t5_w1th1n_cJfnX3Ly4DxoWd5g}
```

Voila.
