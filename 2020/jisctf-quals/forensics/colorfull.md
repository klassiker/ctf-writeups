# Colorfull

## Task

Our team capture this traffic. We believe that there is some malicious activity inside it. Can you figure out it??

File: colurfull.pcapng

## Solution

We open this file with wireshark.

After looking at UDP and DNS we continue with `http or http2 or ftp or ftp-data`.

There is an interesting packet there: `STOR ./files.zip`

We follow the TCP stream and save the data as `RAW` to `files.zip`.

The file is password protected so we use `john` to crack it:

```bash
$ zip2john files.zip > hash
$ john --wordlist=rockyou.txt hash
$ john --show hash
files.zip/secret_data.txt:labeba:secret_data.txt:files.zip:files.zip

1 password hash cracked, 0 left
```

After extracting with `labeba` we get `secret_data.txt`.

```bash
$ head -10 secret_data.txt
34-177-76
34-177-76
34-177-76
34-177-76
34-177-76
34-177-76
34-177-76
34-177-76
34-177-76
34-177-76
```

Took me some time to realize that these are RGB values, therefore the challenge name `colorfull`.

```python
from PIL import Image
import pytesseract

with open("secret_data.txt", "r") as f:
    d = f.read().split("\n")
    d.pop()
    d = [int(x) for y in d for x in y.split("-")]


for x in range(280,300):
    width = x
    height = 200
    c = d.copy()
    for x in range(len(d), width*height*3):
        c.append(0)
    im = Image.frombytes("RGB", (width, height), bytes(c))
    txt = pytesseract.image_to_string(im)
    if len(txt) > 3:
        print(width)
        im.resize((1000,1000)).save("imgs/img_{}.png".format(width))
```

We read the values into a flat array, bruteforce the width and use a constant height. We fill the array up with black pixels at the end to compensate for that.

We then create an image from bytes and use `pytesseract` to check if there is text in the image. If so, a resized version will be saved into `imgs`.

```bash
$ python image.py
289
```

And we can read the flag: `JISCTF{EF1LT3R4T3D_D4T4_1N_1M4G3_F1L3}`
