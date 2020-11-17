# Fibonacci

## Task

File: Fibonacci.7z

## Solution

We can't unzip that file. The magic header is broken.

```bash
$ file Fibonacci.7z
Fibonacci.7z: data
$ xxd Fibonacci.7z | head
00000000: 4242 4242 3742 7abc 42af 271c 0042 04c9  BBBB7Bz.B.'..B..
00000010: 336d e569 0542 0000 0000 0000 5a00 0000  3m.i.B......Z...
00000020: 0000 4200 0077 9ed7 14e0 0be2 0561 5d00  ..B..w.......a].
00000030: 2168 b490 09c2 6442 c064 0461 7378 6c59  !h....dB.d.asxlY
00000040: 041b 7ec3 e075 08cf 3b81 a186 1f0c 2557  ..~..u..;.....%W
00000050: fded 72e7 04b6 20eb a242 dcc7 e2f1 ee76  ..r... ..B.....v
00000060: 407f 6fa8 09dc e8db 19d4 ea35 0743 14a5  @.o........5.C..
00000070: e962 5c96 b485 329f e0ea 4c10 fc8f 4493  .b\...2...L...D.
00000080: 5407 0631 066e 7938 ebc2 7e52 a17f a547  T..1.ny8..~R...G
00000090: 4246 6502 189a ce8c 6ae2 469c 8f93 0f57  BFe.....j.F....W
```

Correct one would be: `37 7A BC AF 27 1C`. Just fixing that doesn't help.

So, why is this called `Fibonacci`? If you keep that in mind this is pretty straight forward.

If the byte position is in a gap of the sequence we keep it.
1 -> 1, drop
1 -> 2, drop
2 -> 3, drop
3 -> 5, keep 4
5 -> 8, keep 6 and 7

This is my very bad script, using an intermediate boolean array:

```python2
fibs = [1,1,2,3,5,8,13,21,34,55,89,144,233,377,610,987,1597,2584,4181,6765]
dec = []

prev = 0
for x in fibs:
    for y in range(x - prev - 1):
        dec.append(1)
    dec.append(0)
    prev = x

f = open("fib.7z", "rb")
d = f.read()
f.close()

f = open("dec.7z", "w")
for b in range(len(d)):
    if dec[b]:
        f.write(d[b])

f.close()
```

After extracting we have a `Fibonacii` file. Flag is in the last line:

`"AFFCTF{Hitchhiker}," said Deep Thought, with infinite majesty and calm.”`
