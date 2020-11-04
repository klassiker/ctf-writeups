# Generic Flag Checker 2

## Task

Flag Checker Industries™ is back with another new product, the Generic Flag Checker® Version 2℠! This time, the flag has got some foolproof math magic preventing pesky players from getting the flag so easily.

File: gfc2

## Solution

```bash
$ file gfc2
gfc2: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=453a5fd2afeb8a872bfa82de41ba955683428c69, for GNU/Linux 3.2.0, stripped
```

We start off with `radare2`. We see a lot happening in `main`. In the lower section:

`0x00001250      e8dbfdffff     call sym.imp.strncmp`

I was pretty lazy and just used gdb with a breakpoint.

The binary is stripped and we can't access `main` directly.

```nasm
gdb-peda$ info functions
All defined functions:

Non-debugging symbols:
0x0000000000001030  strncmp@plt
0x0000000000001040  fmemopen@plt
0x0000000000001050  __isoc99_fscanf@plt
0x0000000000001060  puts@plt
0x0000000000001070  fclose@plt
0x0000000000001080  __stack_chk_fail@plt
0x0000000000001090  fgets@plt
0x00000000000010a0  fprintf@plt
0x00000000000010b0  fseek@plt
```

`radare2` told us that main is at `0x000010c0`.

After starting the binary and aborting the `fgets` call we have the real address.

```nasm
gdb-peda$ info functions fseek@plt
All functions matching regular expression "fseek@plt":

Non-debugging symbols:
0x00005555555550b0  fseek@plt
```

So we set our breakpoint at `0x0000555555555250` because that is the offset from the call to `strncmp` in our main function.

```nasm
gdb-peda$ b *0x0000555555555250
Breakpoint 1 at 0x555555555250
gdb-peda$ r
Starting program: gfc2
what's the flag?
i dunno
Breakpoint 1, 0x0000555555555250 in ?? ()
gdb-peda$ info register rsi
rsi            0x7fffffffe1a0      0x7fffffffe1a0
gdb-peda$ x/1 0x7fffffffe1a0
0x7fffffffe1a0: "nactf{s0m3t1m3s_dyn4m1c_4n4lys1s_w1n5_gKSz3g6RiFGkskXx}"
```
