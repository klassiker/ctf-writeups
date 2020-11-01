# Steg Ultimate

## Task

You reached the final stage, can you unravel your way out of this one?

File: stegultimate.jpg

## Solution

Well, this was three times the points of `What's the password` so I didn't expect the first step to be `steghide` without password.

```bash
$ steghide info stegultimate.jpg -p ""
"stegultimate.jpg":
  format: jpeg
  capacity: 62,4 KB
  embedded file "steg3.jpg":
    size: 37,2 KB
    encrypted: rijndael-128, cbc
    compressed: yes
$ steghide extract -sf stegultimate.jpg -xf steg3.jpg -p ""
wrote extracted data to "steg3.jpg".
```

So, now we have `steg3.jpg`. Now we have to use another tool, right?

```bash
$ steghide info steg3.jpg -p ""
"steg3.jpg":
  format: jpeg
  capacity: 2,0 KB
  embedded file "steganopayload473955.txt":
    size: 29,0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes
$ steghide extract -sf steg3.jpg -xf payload.txt -p ""
wrote extracted data to "payload.txt".
$ cat payload.txt
https://pastebin.com/YnKqT9s3
```

We find a `Hmmmm. What cipher is this? Sometimes, it's not the type we think.` and base64.

```bash
$ cat base64.txt | base64 -d | file -
/dev/stdin: PNG image data, 652 x 88, 8-bit/color RGBA, non-interlaced
```

It contains the flag: `CYCTF{2_f0r_th3_pr1c3_0f_1_b64}`
