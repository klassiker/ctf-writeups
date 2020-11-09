# speedrun-16

## Task

nc chal.2020.sunshinectf.org 30016

File: chall_16

Tags: binary ananlysis

## Solution

This `main` looks a lot like `speedrun-09`. We XOR each char with the `range(0x30,0x94)` and compare it to the chars of `Queue epic guitar solo *syn starts shredding*\n`.

If you code it and execute it... all the XORing does nothing. Just enter the key and get the flag.

```bash
$ nc chal.2020.sunshinectf.org 30016
Queue epic guitar solo *syn starts shredding*
ls
chall_16
flag.txt
cat flag.txt
sun{beast-and-the-harlot-73058b6d2812c771}
```
