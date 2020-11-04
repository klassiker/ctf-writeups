# Numbers

## Task

What do the numbers mean?

File: flag.txt

## Solution

Inside the file:

`111 98 100 117 103 124 98 116 100 50 50 96 89 67 53 83 68 83 54 126`

From experience I can see that the first three chars are obd, which happens to be at offset +1 from `nac`, the flag format.

```python
>>> ''.join([chr(int(x) - 1) for x in s.split(" ")])
'nactf{asc11_XB4RCR5}'
```
