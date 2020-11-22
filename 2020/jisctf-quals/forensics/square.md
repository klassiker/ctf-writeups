# SQUARE

## Task

Our Information Security Center team noticed something strange over the network. Can you figure out what happened?

You can download challenge file from this link :

https://easyupload.io/2xqnm4

## Solution

We open `wireshark` and first look at `udp`.

We find some funny queries to `000100111010100101010101010.jordaninfosec.org` and others like it.

We filter all the queries containing that domain: `dns.flags == 0x100 and dns.qry.name contains "jordan"`

Okay, we have filtered them now. The easiest way to export them is by using `tshark`:

```bash
$ tshark -r mysquare.pcapng -Tfields -e 'dns.qry.name' 'dns.flags == 0x100 and dns.qry.name contains "jordan"'
jordaninfosec.org
jordaninfosec.org
jordaninfosec.org
99567332698.jordaninfosec.org
00000000000000111111110011000011111100000000000000.jordaninfosec.org
00111111111100110000111100111100111100111111111100.jordaninfosec.org
00110000001100110011000011110011001100110000001100.jordaninfosec.org
00110000001100111100111111111100001100110000001100.jordaninfosec.org
00110000001100110000111111110011001100110000001100.jordaninfosec.org
00111111111100110011001100001111111100111111111100.jordaninfosec.org
00000000000000110011001100110011001100000000000000.jordaninfosec.org
11111111111111110011000000001100111111111111111111.jordaninfosec.org
00001100111100001111111100110000111100000011000011.jordaninfosec.org
00001100000011000000110000001111000000001100111100.jordaninfosec.org
11001111111100001111110000001111000011110000110011.jordaninfosec.org
00110011000011001100001111001100000011000011110011.jordaninfosec.org
11001111000000001111001111111100000000110011001111.jordaninfosec.org
11110000000011000011111111000000110000000000111111.jordaninfosec.org
00110000111100000011110011111100000000111111110011.jordaninfosec.org
11001100001111110000000000111100111111001100110000.jordaninfosec.org
00000000001100110000001100001100000000000000110000.jordaninfosec.org
11111111111111110011001100110000001111110000110000.jordaninfosec.org
00000000000000110011000000111111001100110011001100.jordaninfosec.org
00111111111100111100110011110011001111110011001100.jordaninfosec.org
00110000001100111111111111111111000000000011110000.jordaninfosec.org
00110000001100110000000011000011111111111100110011.jordaninfosec.org
00110000001100111111111111000011110011000000111100.jordaninfosec.org
00111111111100110011111100001111110000110011001100.jordaninfosec.org
00000000000000110000000011001100111100001111111100.jordaninfosec.org
```

It took me some intense staring at the screen to recognize the pattern here. It's an QRCode.

After extracting only `1`s and `0`s to a file I wrote this script:

```python
from PIL import Image
import pytesseract

with open("qr.txt", "r") as f:
    d = f.read().split("\n")
    d.pop()
    j = ''.join(d)
    c = [(0,0,0) if j[x:x+2] == "00" else (255,255,255) for x in range(0, len(j), 2)]
    f = [v for s in c for v in s]

width = 25
height = 25
im = Image.frombytes("RGB", (width, height), bytes(f))
im.resize((1000,1000)).save("qr.png")
```

`00` -> `black`

`11` -> `white`

I just like to resize stuff...

After that we can easily read the code:

```bash
$ zbarimg qr.png --quiet --raw
JISCTF{D0KX_4R3_4RK1V3D_F1L3S}
```
