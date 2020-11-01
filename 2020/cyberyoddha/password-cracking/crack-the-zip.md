# Crack the Zip!

## Task

I'm not able to get into this zip file ...

File: flag.zip

## Solution

We use john to crack this one:

```bash
$ zip2john flag.zip > zip.hashes
$ john --wordlist=rockyou.txt zip.hashes
$ john --show zip.hashes
flag.zip/flag.txt:not2secure:flag.txt:flag.zip::flag.zip
```
