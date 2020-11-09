# speedrun-09

## Task

nc chal.2020.sunshinectf.org 30009

File: chall_09

Tags: binary analysis

## Solution

We open this file in `radare2` and take a look at what's happening.

```nasm
[0x000006c0]> afl
0x000006c0    1 42           entry0
0x000006f0    4 50   -> 40   sym.deregister_tm_clones
0x00000730    4 66   -> 57   sym.register_tm_clones
0x00000780    5 58   -> 51   sym.__do_global_dtors_aux
0x000007c0    1 10           entry.init0
0x00000920    1 2            sym.__libc_csu_fini
0x00000924    1 9            sym._fini
0x000008b0    4 101          sym.__libc_csu_init
0x000007ca    1 19           sym.win
0x00000680    1 6            sym.imp.system
0x000007dd   10 202          main
0x00000630    3 23           sym._init
0x00000660    1 6            sym.imp.strlen
0x00000670    1 6            sym.imp.__stack_chk_fail
0x00000000    2 25           loc.imp._ITM_deregisterTMCloneTable
0x00000690    1 6            sym.imp.fgets
0x000006a0    1 6            sym.imp.exit
[0x000006c0]> pdf@main
            ; DATA XREF from entry0 @ 0x6dd
┌ 202: int main (int argc, char **argv, char **envp);
│           ; var int64_t var_54h @ rbp-0x54
│           ; var char *s @ rbp-0x50
│           ; var int64_t canary @ rbp-0x18
│           0x000007dd      55             push rbp
│           0x000007de      4889e5         mov rbp, rsp
│           0x000007e1      53             push rbx
│           0x000007e2      4883ec58       sub rsp, 0x58
│           0x000007e6      64488b042528.  mov rax, qword fs:[0x28]
│           0x000007ef      488945e8       mov qword [canary], rax
│           0x000007f3      31c0           xor eax, eax
│           0x000007f5      488b15640820.  mov rdx, qword [obj.stdin]  ; obj.stdin__GLIBC_2.2.5
│                                                                      ; [0x201060:8]=0 ; FILE *stream
│           0x000007fc      488d45b0       lea rax, [s]
│           0x00000800      be31000000     mov esi, 0x31               ; '1' ; int size
│           0x00000805      4889c7         mov rdi, rax                ; char *s
│           0x00000808      e883feffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
│           0x0000080d      488d45b0       lea rax, [s]
│           0x00000811      4889c7         mov rdi, rax                ; const char *s
│           0x00000814      e847feffff     call sym.imp.strlen         ; size_t strlen(const char *s)
│           0x00000819      4889c3         mov rbx, rax
│           0x0000081c      488d3dfd0720.  lea rdi, obj.key            ; 0x201020 ; "y\x17FU\x10S_]U\x10XUBU\x10D_:" ; const char *s
│           0x00000823      e838feffff     call sym.imp.strlen         ; size_t strlen(const char *s)
│           0x00000828      4839c3         cmp rbx, rax
│       ┌─< 0x0000082b      755e           jne 0x88b
│       │   0x0000082d      c745ac000000.  mov dword [var_54h], 0
│      ┌──< 0x00000834      eb32           jmp 0x868
│      ││   ; CODE XREF from main @ 0x87d
│     ┌───> 0x00000836      8b45ac         mov eax, dword [var_54h]
│     ╎││   0x00000839      4898           cdqe
│     ╎││   0x0000083b      0fb64405b0     movzx eax, byte [rbp + rax - 0x50]
│     ╎││   0x00000840      83f030         xor eax, 0x30
│     ╎││   0x00000843      89c1           mov ecx, eax
│     ╎││   0x00000845      8b45ac         mov eax, dword [var_54h]
│     ╎││   0x00000848      4863d0         movsxd rdx, eax
│     ╎││   0x0000084b      488d05ce0720.  lea rax, obj.key            ; 0x201020 ; "y\x17FU\x10S_]U\x10XUBU\x10D_:"
│     ╎││   0x00000852      0fb60402       movzx eax, byte [rdx + rax]
│     ╎││   0x00000856      38c1           cmp cl, al
│    ┌────< 0x00000858      740a           je 0x864
│    │╎││   0x0000085a      bf00000000     mov edi, 0                  ; int status
│    │╎││   0x0000085f      e83cfeffff     call sym.imp.exit           ; void exit(int status)
│    │╎││   ; CODE XREF from main @ 0x858
│    └────> 0x00000864      8345ac01       add dword [var_54h], 1
│     ╎││   ; CODE XREF from main @ 0x834
│     ╎└──> 0x00000868      8b45ac         mov eax, dword [var_54h]
│     ╎ │   0x0000086b      4863d8         movsxd rbx, eax
│     ╎ │   0x0000086e      488d3dab0720.  lea rdi, obj.key            ; 0x201020 ; "y\x17FU\x10S_]U\x10XUBU\x10D_:" ; const char *s
│     ╎ │   0x00000875      e8e6fdffff     call sym.imp.strlen         ; size_t strlen(const char *s)
│     ╎ │   0x0000087a      4839c3         cmp rbx, rax
│     └───< 0x0000087d      72b7           jb 0x836
│       │   0x0000087f      488d3dae0000.  lea rdi, str.bin_sh         ; 0x934 ; "/bin/sh" ; const char *string
│       │   0x00000886      e8f5fdffff     call sym.imp.system         ; int system(const char *string)
│       │   ; CODE XREF from main @ 0x82b
│       └─> 0x0000088b      90             nop
│           0x0000088c      488b45e8       mov rax, qword [canary]
│           0x00000890      644833042528.  xor rax, qword fs:[0x28]
│       ┌─< 0x00000899      7405           je 0x8a0
│       │   0x0000089b      e8d0fdffff     call sym.imp.__stack_chk_fail ; void __stack_chk_fail(void)
│       │   ; CODE XREF from main @ 0x899
│       └─> 0x000008a0      4883c458       add rsp, 0x58
│           0x000008a4      5b             pop rbx
│           0x000008a5      5d             pop rbp
└           0x000008a6      c3             ret
```

We have a `win` function again but also a call to `system` in our `main` function. It's easier to understand this function if you open it up in the visual mode `VV @ main`.

We read in a string with `fgets`, compare the `strlen` to the `strlen` of `obj.key`. If they are equal we initialize `var_54h` with 0 and jump to a loop.

This loop iterates over the chars until `var_54h` is above or equal to the `strlen` of `obj.key`. After that it calls `system`.

Each input char is XORed with `0x30` and compared to the char of the key. If they match `var_54h` is incremented.

We write this easy python one-liner:

```python
>>> ''.join([chr(x ^ 0x30) for x in b'y\x17FU\x10S_]U\x10XUBU\x10D_:'])
"I've come here to\n"
```

Let's enter that on the remote:

```bash
$ nc chal.2020.sunshinectf.org 30009
I've come here to
ls
chall_09
flag.txt
cat flag.txt
sun{coming-home-4202dcd54b230a00}
```
