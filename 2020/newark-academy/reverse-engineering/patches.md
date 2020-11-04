# Patches

## Task

This binary does nothing.

File: patches

## Solution

This is not the intended solution. I was supposed to patch the binary. What I did was this:

```nasm
gdb-peda$ info functions
All defined functions:

Non-debugging symbols:
0x0000000000001000  _init
0x0000000000001030  puts@plt
0x0000000000001040  __stack_chk_fail@plt
0x0000000000001050  _start
0x0000000000001261  print_flag
0x00000000000012d9  main
0x00000000000012f0  __libc_csu_init
0x0000000000001360  __libc_csu_fini
0x0000000000001368  _fini
gdb-peda$ disass main
Dump of assembler code for function main:
   0x00000000000012d9 <+0>:     push   rbp
   0x00000000000012da <+1>:     mov    rbp,rsp
   0x00000000000012dd <+4>:     lea    rdi,[rip+0xeac]        # 0x2190
   0x00000000000012e4 <+11>:    call   0x1030 <puts@plt>
   0x00000000000012e9 <+16>:    mov    eax,0x0
   0x00000000000012ee <+21>:    pop    rbp
   0x00000000000012ef <+22>:    ret
End of assembler dump.
```

Expecting `print_flag` to do what it's name suggests I just set a breakpoint at the `ret` instruction and modified `rip`.

```nasm
gdb-peda$ disass main
Dump of assembler code for function main:
   0x00005555555552d9 <+0>:     push   rbp
   0x00005555555552da <+1>:     mov    rbp,rsp
   0x00005555555552dd <+4>:     lea    rdi,[rip+0xeac]        # 0x555555556190
   0x00005555555552e4 <+11>:    call   0x555555555030 <puts@plt>
   0x00005555555552e9 <+16>:    mov    eax,0x0
   0x00005555555552ee <+21>:    pop    rbp
   0x00005555555552ef <+22>:    ret
gdb-peda$ b *0x00005555555552ef
Breakpoint 1 at 0x5555555552ef
gdb-peda$ r
Starting program: patches
Goodbye.
Breakpoint 1, 0x00005555555552ef in main ()
gdb-peda$ info functions print_flag
All functions matching regular expression "print_flag":

Non-debugging symbols:
0x0000555555555261  print_flag
gdb-peda$ set $rip = 0x0000555555555261
gdb-peda$ s
0x0000555555555262 in print_flag ()
gdb-peda$ c
Continuing.
nactf{unl0ck_s3cr3t_funct10n4l1ty_w1th_b1n4ry_p4tch1ng_L9fcKhyPupGVfCMZ}
```

Insert shrug here.
