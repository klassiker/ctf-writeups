# What's the password?

## Task

My friend gave me this file, he said there was something hidden

File: sudo.jpg

## Solution

The challenge name gave it away, we need to use `steghide` to extract the data. Normally I would build a wordlist from the challenge description but trying `sudo` succeeded.

```bash
$ steghide extract -sf sudo.jpg -p sudo -xf "steganopayload457819.txt"
wrote extracted data to "steganopayload457819.txt".
$ cat steganopayload457819.txt
CYCTF{U$3_sud0_t0_achi3v3_y0ur_dr3@m$!}
```
