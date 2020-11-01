# Hashfun

## Task

I guess there is no way to recover the flag.

File: generate.py

## Solution

We have the following script:

```python
def hashfun(msg):
    digest = []
    for i in range(len(msg) - 4):
        digest.append(ord(msg[i]) ^ ord(msg[i + 4]))
    return digest

# [10, 30, 31, 62, 27, 9,  4,  0,  1,  1, 4,  4,  7, 13,  8, 12, 21, 28, 12,  6, 60]
```

We now just need to reverse it. Since no unxored character is presented we have to guess the output starts with the flag format `SCR{` which happens to be 4 characters long, exactly what we need.

```python
def revHashfun(msg):
    out = ["" for x in range(len(msg) + 4)]
    out[:4] = "CSR{"
    for x in range(len(msg)):
        out[x + 4] = chr(msg[x] ^ ord(out[x]))
    return ''.join(out)
```

Flag is: `CSR{IMMERDIESEMATHEMATIK}`
