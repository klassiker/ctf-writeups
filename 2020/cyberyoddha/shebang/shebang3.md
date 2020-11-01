# Shebang 3

## Task

These files are the same...

## Solution

After doing a diff we find some curly braces and single chars among hundreds of encoding failures.

```bash
$ grep '^.$' file2.txt | tr -d '\n';echo
1CYCTF{SPOT_TH3_D1FF}ajb73pkcsduole92yxtnmgfQD650rqih_ZROB?84.+$!
```
