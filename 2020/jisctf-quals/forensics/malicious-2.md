# Malicious 2

## Task

File: mycv.docx

## Solution

```bash
$ file mycv.docx
mycv.docx: CDFV2 Encrypted
```

We can use `office2john` to crack it:

```bash
$ office2john mycv.docx > hash
$ john --wordlist=~/ctf-tools/rockyou.txt hash
$ john --show hash
mycv.docx:princess101

1 password hash cracked, 0 left
```

We can use this python tool to decrypt the file: https://github.com/nolze/msoffcrypto-tool

After that we use `binwalk`:

```bash
$ binwalk -e dec.docx
$ grep -Ro 'JISCTF{[^}]*}' _dec.docx.extracted/
_dec.docx.extracted/word/document.xml:JISCTF{H4PPY_HUNT1NG}
```
