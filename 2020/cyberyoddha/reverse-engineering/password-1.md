# Password 1

## Task

We uncovered the first password encryption file. Can you crack it for us?

File: password1.py

## Solution

We are given a python script that checks the input password by array indexing and comparison to chars. So we just use a regular expression to convert this to assignments:

```bash
$ sed 's|\(password\[[^]]*\]\) == \(.*\) and|\1 = \2|g' password1.py -i
```

We fix the small errors by hand and create an array with the required length, `''.join(password)` and we have it.
