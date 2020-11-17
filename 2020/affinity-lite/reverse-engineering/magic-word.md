# Magic Word

## Solution

We need a magic word to get the response here, can you discover it?

File: magicword

## Solution

We analyze the binary using `radare2`. There are a lot of `c++` functions and calls in `main`.

We also have this at the start:

```nasm
│           0x00002d90      c78550ffffff.  mov dword [var_b0h], 0x54   ; 'T'
│           0x00002d9a      c78554ffffff.  mov dword [var_ach], 0x6f   ; 'o'
│           0x00002da4      c78558ffffff.  mov dword [var_a8h], 0x74   ; 't'
│           0x00002dae      c7855cffffff.  mov dword [var_a4h], 0x61   ; 'a'
│           0x00002db8      c78560ffffff.  mov dword [var_a0h], 0x6c   ; 'l'
│           0x00002dc2      c78564ffffff.  mov dword [var_9ch], 0x6c   ; 'l'
│           0x00002dcc      c78568ffffff.  mov dword [var_98h], 0x79   ; 'y'
│           0x00002dd6      c7856cffffff.  mov dword [var_94h], 0x52   ; 'R'
│           0x00002de0      c78570ffffff.  mov dword [var_90h], 0x61   ; 'a'
│           0x00002dea      c78574ffffff.  mov dword [var_8ch], 0x6e   ; 'n'
│           0x00002df4      c78578ffffff.  mov dword [var_88h], 0x64   ; 'd'
│           0x00002dfe      c7857cffffff.  mov dword [var_84h], 0x6f   ; 'o'
│           0x00002e08      c745806d0000.  mov dword [var_80h], 0x6d   ; 'm'
│           0x00002e0f      c74584430000.  mov dword [var_7ch], 0x43   ; 'C'
│           0x00002e16      c74588680000.  mov dword [var_78h], 0x68   ; 'h'
│           0x00002e1d      c7458c610000.  mov dword [var_74h], 0x61   ; 'a'
│           0x00002e24      c74590720000.  mov dword [var_70h], 0x72   ; 'r'
│           0x00002e2b      c74594730000.  mov dword [var_6ch], 0x73   ; 's'
```

Actually reversing this using ghidra would take way too much time. There is a function `sym.returnFlag_abi:cxx11` using a `ThisIsNotIt` in a call to `sym.decrypt__std::...` which calls `sym.decrypt_vigenere`.... let's not do that.

Entering `TotallyRandomChars` after executing the binary we get the flag.
