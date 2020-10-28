# Rhazes

## Task

Find the hidden chemical formula in this depiction of an alchemist known as Rhazes.

File: Rhazes.jpg

Tags: steganography

## Solution

The file is huge (10MB) and there is nothing interesting to find using `file`, `foremost`, `binwalk`, `stegsolve`. I used a patched version of `stegsolve` to allow zoom this very large image:

```bash
$ file Rhazes.jpg
Rhazes.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 72x72, segment length 16, baseline, precision 8, 8536x9688, components 3
```

Loading was very slow and I didn't find anything. I then tried `steghide` with a wordlist generated from the challenge task description. `Rhazes` was the correct password.

```bash
$ steghide info Rhazes.jpg
"Rhazes.jpg":
  format: jpeg
  capacity: 481,2 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase:
  embedded file "MagicSanitizer.jpg":
    size: 407,0 KB
    encrypted: rijndael-128, cbc
    compressed: yes
$ steghide extract -p Rhazes -sf Rhazes.jpg -xf MagicSanitizer.jpg
$ file MagicSanitizer.png
MagicSanitizer.png: PNG image data, 4335 x 350, 8-bit/color RGBA, non-interlaced
```

After extracting don't let the filename fool you. A trying `zsteg` I viewed the file and saw a weird pattern that seemed to be displaced in the x-direction. So I opened `stegsolve` again and used the `Stereogram solver`. Yes, you need to click a lot since the file is `4335` pixels wide.

At offset `161` we can see the flag on a black background with letters in rainbow colors.

`RaziCTF{C2H5OH_k1ll5_54r5_c0v_2}`
