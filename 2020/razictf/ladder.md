# Ladder

## Task

You must attack all enemy BASEs, start low and move your way up.

File: enc.txt (in Ladder.zip)

Tags: cryptography

## Solution

The file contains base32. We can assume it's not just b64decode(b32decode(content)). Thankfully there is a tool called `basecrack`:  https://github.com/mufeedvh/basecrack

```bash
$ cat enc.txt | python basecrack.py -m
[>] Enter Encoded Base:
[-] Iteration: 1

[-] Heuristic Found Encoding To Be: Base32

[-] Decoding as Base32: 3ujJnYq1YPYVouAmFC24c36quqhmGkcm5LL5mZGvs6w4mewG9Cg3jLhWJgcdMyT1EM1ZYp2cCabt6syu9oAnjLeoDkiJKgS7

[-] Iteration: 2

[-] Heuristic Found Encoding To Be: Base58

[-] Decoding as Base58: B3ubcEe7waVuE55z96FTrO8JvfuevhFXEhXqzkWINOiIaRd4OlGLhL5jgqUXaQRwNC0CXl

[-] Total Iterations: 2

[-] Encoding Pattern: Base32 -> Base58

[-] Magic Decode Finished With Result: B3ubcEe7waVuE55z96FTrO8JvfuevhFXEhXqzkWINOiIaRd4OlGLhL5jgqUXaQRwNC0CXl

[-] Finished in 0.0018 seconds
```

Hm, maybe I was wrong? Let's try it with CyberChef.

Just use all `From Base` tools available under the section `Data format`.

That works and we get the flag: `RaziCTF{w3_G0t_4ll_tHe_Ba$3s}`

But why did basecrack fail? GitHub page states: `Decode Base16, Base32, Base36, Base58, Base62, Base64, Base64Url, Base85, Base91, Base92`

Turns out there is an issue with base62: https://github.com/mufeedvh/basecrack/issues/4

I fixed that locally, still no luck.

`Encoding Pattern: Base32 -> Base58 -> Base62 -> Base64 -> Base92`

We are missing Base85 here. Debugging the script I found this:

`bad base85 character at position 36` - this is a `/`

As it turns out there is Base85 and Ascii85 and Ascii85 is not supported. After adding this (it's contained in the python base64 module) it works.

```
[-] Encoding Pattern: Base32 -> Base58 -> Base62 -> Base64 -> Ascii85

[-] Magic Decode Finished With Result: RaziCTF{w3_G0t_4ll_tHe_Ba$3s}
```
