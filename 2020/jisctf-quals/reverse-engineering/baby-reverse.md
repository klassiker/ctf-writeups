# Baby Reverse

## Task

File: Easy.zip

## Solution

After looking the the file with `radare2` I noticed a lot of strings being moved around and inserted in the flag format.

Instead of looking at it for long I just used dynamic analysis to let the binary do it's thing.

There are some checks in place that we need to circumvent. At each jump that follows a comparison we set a breakpoint. It it doesn't jump where we want it to jump we modify `$rip`.

```bash
gdb-peda$ b *main+47
Breakpoint 1 at 0x769
gdb-peda$ b *main+789
Breakpoint 3 at 0xa4f
```

We also set a breakpoint after the last loop that loaded all the correct chars.

After reaching the first breakpoint:

```bash
gdb-peda$ set $rip = 0x55555555477c
```

At the second breakpoint:

```bash
gdb-peda$ x/s $rbp-0x40
0x7fffffffe050: "JISCTF{Th1S_1S_4N_e4Sy_R3v3Rs1Ng}"
```
