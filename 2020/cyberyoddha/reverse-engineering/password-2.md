# Password 2

## Task

You did so well on the last one, can you try this?

File: password2.py

## Solution

This one looks harder but is actually faster to solve:

```python
def checkPassword(password):
    if(len(password) != 47):
      return False
    newPass = list(password)
    for i in range(0,9):
      newPass[i] = password[i]
    for i in range(9,24):
      newPass[i] = password[32-i]
    for i in range(24,47,2):
      newPass[i] = password[70-i]
    for i in range(45,25,-2):
      newPass[i] = password[i]
    password = "".join(newPass);
    return password == "CYCTF{ju$@rcs_3l771l_@_t}bd3cfdr0y_u0t__03_0l3m"
```

It just changes the character order and a `return password` for `checkPassword("CYCTF{ju$@rcs_3l771l_@_t}bd3cfdr0y_u0t__03_0l3m")` gives us the flag.
