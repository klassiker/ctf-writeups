# Oligar's Tricky RSA

## Task

The crypto master Oligar just sent this file with three numbers. What do they mean?

File: rsa.txt

## Solution

We are given `c`, `e` and `n`. So let's put `n` into http://factordb.com/index.php:

```python
c = 97938185189891786003246616098659465874822119719049
e = 65537
n = 196284284267878746604991616360941270430332504451383

# From http://factordb.com/index.php?query=196284284267878746604991616360941270430332504451383
q = 10252256693298561414756287
p = 19145471103565027335990409
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
print(binascii.unhexlify(hex(pow(c, d, n))[2:]).decode("utf-8"))
```

We get the flag: `nactf{sn3aky_c1ph3r}`
