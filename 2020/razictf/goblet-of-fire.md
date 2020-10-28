# Goblet of Fire

## Task

I think Granger is smart enough to help you solve this challenge!

File: Goble of Fire.txt

Tags: steganography

## Solution

I have a setting in nano that shows me inconsisten use of tabs and spaces. This showed me a weird pattern in the file. I tried extracting it manually with some piping magic but no luck.

After researching a few minutes I found `snow`. Also called `stegsnow`. Used to hide data in files with tabs and spaces. Compression and encryption is optional.

After trying with and without compression I used `Granger` as the password:

```bash
$ ./snow -p Granger Goblet\ of\ Fire.txt;echo
RaziCTF{175_ju57_tabs_4nd_5p4c35}
```
