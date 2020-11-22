# Baby Crypto

## Task

File: Baby_crypto.zip

## Solution

In the file the flag bytes are XORed with random values, the seed is also XORed, but with a constant value `0x99`. We can easily XOR it again, extract the time, seed our PRNG with it, generate the random values and XOR the rest.

```python
import random
import sys
import time

# This was given
#
# ct = str(time.time()).encode('ASCII')
# random.seed(ct)
# flag = 'data_here'.encode('ASCII')
# k1 = [random.randrange(256) for _ in flag]
# ciphertext = [m ^ k for (m,k ) in zip(flag + ct, k1 + [0x99]*len(ct))]
#
# with open(sys.argv[1], "wb") as f:
#     f.write(bytes(ciphertext))

f = open("flag.enc", "rb")
d = f.read()
f.close()

o = []
for c in d:
    o.append(chr(c ^ 0x99))

x = len(str(time.time()))
ct = ''.join(o[len(o)-x:]).encode("ASCII")
random.seed(ct)
flag = "A"*(len(o)-len(ct))
k1 = [random.randrange(256) for _ in flag]
unciphertext = [m ^ k for (m,k) in zip(d, k1 + [0x99]*len(ct))]
print(''.join([chr(x) for x in unciphertext]))
```

```bash
$ python baby.py
JISCTF{B4BY_ENCRYPT10N_JISCTF2020_QUALIFICATION_RND_101}1605733600.6308804
```
