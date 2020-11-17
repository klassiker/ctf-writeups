# TODO

## Task

On our victim's computer, there was some old script. It looks like someone left the message and the unfinished function used to read the content of that message. We have to obtain the message, could you do that?

File: task.py

## Solution

Task is this:

```python
#!/usr/bin/env python3

def encode(msg):
    output = ''

    for i in range(len(msg)):
        temp = ord(msg[i]) * 0x40
        temp = temp >> 4
        if 0xc0 <= temp < 0xe8:
            output = str(int(msg[i]) * 0x1234) + output
        else:
            output = chr(ord(msg[i]) * 0x10) + output

    return output


# TODO implement the decode function
def decode(msg):
    raise NotImplementedError


def shift(msg):
    j = len(msg) - 1
    output = ''

    for i in range(len(msg)//2):
        output += msg[i] + msg[j]
        j -= 1

    return output


# TODO implement the unshift function
def unshift(msg):
    raise NotImplementedError


if __name__ == '__main__':
    # shifted = shift('<REDACTED>')
    # hashed = encode(shifted)
    hashed = '46604660ڀ٠װװސ23300۰ސݐ18640ܠݰװۀڠ18640۰ؠѠȐՀȐа4660ѠȐѠߐА'
    # CODE HERE
```

The `unshift` is easy to implement. Odd char positions are prepended at the end of the output, even position are append at the start.

The `decode` is more tricky. We have to search for possible substrings that are valid integers, divide by `0x1234` and check if the result is also a valid integer. Otherwise we simply use `chr(ord(x) // 0x10)`. The string is also reversed.

```python
import string

def unshift(msg):
    out = ["" for x in range(len(msg))]
    for x in range(0,len(msg)//2):
        out[x] = msg[x*2]
        out[len(msg)-x-1] = msg[x*2+1]
    return ''.join([str(x) for x in out])

def decode(msg):
  out = []
  curr = ''
  for c in msg:
    if c in string.digits:
      curr += c
      v = int(curr) // 0x1234
      if v != 0:
        curr = ''
        out.append(v)
    else:
      out.append(chr(ord(c) // 0x10))
  return ''.join([str(x) for x in out])
```

Running it:

```bash
$ python task.py
AFFCTF{4lw4y5_f1n1sh_your_job!!1!}
```
