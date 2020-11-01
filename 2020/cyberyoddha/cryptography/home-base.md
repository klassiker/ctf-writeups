# Home Base

## Task

4a5a57474934325a47464b54475632464f4259474336534a4f564647595653574a354345533454434b52585336564a524f425556435533554e4251574f504a35

## Solution

The given string looks like hex encoded chars. Let's try that:

```python
>>> import binascii
>>> binascii.unhexlify(s)
b'JZWGI42ZGFKTGV2FOBYGC6SJOVFGYVSWJ5CES4TCKRXS6VJROBUVCU3UNBQWOPJ5'
```

Looks like Base32. Let's fire up `basecrack`:

```bash
$ python basecrack.py -m

                python basecrack.py -h [FOR HELP]

[>] Enter Encoded Base: JZWGI42ZGFKTGV2FOBYGC6SJOVFGYVSWJ5CES4TCKRXS6VJROBUVCU3UNBQWOPJ5

[-] Iteration: 1

[-] Heuristic Found Encoding To Be: Base32

[-] Decoding as Base32: NldsY1U3WEppazIuJlVVODIrbTo/U1piQSthag==


[-] Iteration: 2

[-] Heuristic Found Encoding To Be: Base64

[-] Decoding as Base64: 6WlcU7XJik2.&UU82+m:?SZbA+aj


[-] Iteration: 3

[-] Heuristic Found Encoding To Be: Ascii85

[-] Decoding as Ascii85: CYCTF{it5_@_H0m3_2un!}


[-] Iteration: 4

[-] Heuristic Found Encoding To Be: Base92

[-] Decoding as Base92: _ûZI3+_­ÿ\@


[-] Total Iterations: 4

[-] Encoding Pattern: Base32 -> Base64 -> Ascii85 -> Base92

[-] Magic Decode Finished With Result: _ûZI3+_­ÿ\@

[-] Finished in 0.0103 seconds
```

Let's ignore the last iteration.
