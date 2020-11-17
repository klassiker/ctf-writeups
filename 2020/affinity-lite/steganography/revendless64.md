# Revendless64

## Task

File: z.7z

## Solution

Extract the file and:

```bash
$ mv z 0
$ head -c 100 0;echo
==gCTBTR4ZlaWpUYxoFWXpmRTFGRGRXVxI1SW1mSzQVb4dlU6xGSaZFcvJVbGhkUrRWaWNTQ6dVVo92VGpFWkdUMUZFbKdUWtRHM
```

It's a reversed base64 string (and very long).

Let's write a script to decode it 100 times:

```bash
for i in $(seq 0 100);do
  cat "$i" | rev | python2 -c 'import base64;import sys;base64.decode(sys.stdin, sys.stdout)' > "$(($i + 1))"
done
```

We reverse the content and base64decode the contents to the next file.

After that, search for the highest number that still has content:

```bash
$ cat 50
AFFCTF{s1mPle_Rev4s_D0Ne_oNe1}
```
