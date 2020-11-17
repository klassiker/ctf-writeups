# BreakMe

## Task

I encrypted important information and lost my private key!

Can you help me to recover the content of the file?

File: encrypted.txt public.pem

## Solution

We have ciphertext in `encrypted.txt` and a public key in PEM format in `public.pem`.

This is probably an RSA task with a weak public key. This means we can easily find/calculate the prime factors and calculate the private key from them.

We can use `openssl` to extract the information:

```bash
$ openssl rsa -text -noout -pubin -in public.pem
RSA Public-Key: (256 bit)
Modulus:
    00:be:5f:67:0c:7c:df:cc:0b:d3:41:12:d3:bd:71:
    22:9f:d3:e4:46:e5:31:bf:35:16:03:6c:12:58:33:
    6f:6c:51
Exponent: 65537 (0x10001)
```

We now can convert the hex of `Modulus` to an integer and search for that in `factordb`:

```python
>>> int.from_bytes(binascii.unhexlify("00be5f670c7cdfcc0bd34112d3bd71229fd3e446e531bf3516036c1258336f6c51"), "big")
86108002918518428671680621078381724386896258624262971787023054651438740237393
```

After we have found `p` and `q` we can use this script to decrypt the ciphertext:

```python
e = 65537
n = 86108002918518428671680621078381724386896258624262971787023054651438740237393

# From http://factordb.com/index.php?query=86108002918518428671680621078381724386896258624262971787023054651438740237393
q = 286748798713412687878508722355577911069
p = 300290718931931563784555212798489747397
phi = (p - 1) * (q - 1)

def egcd(a, b):
        x,y, u,v = 0,1, 1,0
        while a != 0:
                q, r = b//a, b%a
                m, n = x-u*q, y-v*q
                b,a, x,y, u,v = a,r, u,v, m,n
                gcd = b
        return gcd, x, y

_, d, _ = egcd(e, phi)

import binascii
f = open("encrypted.txt", "rb")
c = int.from_bytes(f.read(), "big")
f.close()
print(binascii.unhexlify("0"+hex(pow(c, d, n))[2:]))
```

Running it:

```bash
$ python dec.py
b'\x02Ca1\xbc\xe8\xad\x165\xe4\xfc\x00AFFCTF{PermRecord}\n'
```
