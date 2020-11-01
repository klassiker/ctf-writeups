# Shebang 1

## Task

This challenge is simple.

## Solution

```bash
$ ls -al
total 300
dr-x------ 1 shebang1 root   4096 Oct 30 07:07 .
drwxr-xr-x 1 root     root   4096 Oct 30 07:07 ..
-rw-r--r-- 1 root     root 298902 Oct  6 22:43 flag.txt
shebang1@c4556110210c:~$ id
uid=1001(shebang1) gid=1001(shebang1) groups=1001(shebang1)
$ cat flag.txt | sort -u | grep ^CYC
CYCTF{w3ll_1_gu3$$_y0u_kn0w_h0w_t0_gr3p}
```
