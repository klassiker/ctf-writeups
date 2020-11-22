# Unknown Ransomware

## Task

Our files were encrypted by an unknown ransomware. Unfortunately, we did not have a binary file from the malware. We will give you one of our encrypted files to try decrypt it. Can you??

File: flag.zip

## Solution

Inside is a `flag.enc` with base64 encoding.

It's very long so probably multiple rounds of base64... I hate that. It's just annoying and no real challenge.

```bash
$ cat flag.enc | base64 -d | base64 -d |base64 -d |base64 -d |base64 -d |base64 -d |base64 -d |base64 -d |base64 -d |base64 -d |base64 -d |base64 -d |base64 -d > decoded.txt
```

Instead of writing a proper script I decided on using multiple pipes until I receive errors...

We receive binary data:

```bash
$ strings decoded.txt | grep -E '[A-Z]{4}'
DNEI
^xTADI
RDHI
```

RDHI? DNEI? Sounds like `IHDR` and `IEND`, things you find in a PNG.

```bash
$ tail -c20 decoded.txt | xxd
00000000: 2502 0000 5244 4849 0d00 0000 0a1a 0a0d  %...RDHI........
00000010: 474e 5089                                GNP.
```

Yeah, PNG file header. Let's reverse it:

```bash
$ xxd -p -c1 decoded.txt | tac | xxd -p -r > file.png
```

And we can read the flag.. backwards...
