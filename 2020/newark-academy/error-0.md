# Error 0

## Task

Rahul has been trying to send a message to me through a really noisy communication channel. Repeating the message 101 times should do the trick!

File: enc.txt

## Solution

We have a file with a lot of `1`s and `0`s. We know the message is repeated 101 times so we split it into 101 parts. Each char consists of 8 bits that form a byte. Now we just need do to a little statistical test to find the most common char that appeared the most over all 101 repetitions.

```python
f = open("enc.txt", "r")
d = f.read().split("\n")[0]
b = len(d)//101
p = [[chr(int(d[x:x+b][y:y+8], 2)) for y in range(0, b, 8)] for x in range(0, len(d), b)]
c = [{} for x in range(len(p[0]))]

for x in range(len(p)):
    for y in range(len(p[x])):
        try:
            c[y][p[x][y]] += 1
        except:
            c[y][p[x][y]] = 1


o = []
for x in range(len(c)):
    highest = 0
    char = ""
    for y in c[x]:
        if c[x][y] > highest:
            char = y
            highest = c[x][y]
    o.append(char)
print(''.join(o))
```

And we get the flag: `nactf{n01sy_n013j_|\|()|$'/}`
