# Shebang 0

## Task

Welcome to the Shebang Linux Series. Here you will be tested on your basic command line knowledge! These challenges will be done threough an ssh connection. Please make all your scripts in the /tmp directory and delete them as soon as you are done to prevent others from finding your scripts. Also please do not try and mess up the challenges on purpose, and report any problems you find to the challenge author. The username is the challenge title, shebang0-6, and the password is the previous challenges flag, but for the first challenge, its shebang0

The first challenge is an introductory challenge. Connect to cyberyoddha.baycyber.net on port 1337 to recieve your flag!

## Solution

```bash
$ ssh cyberyoddha.baycyber.net -p 1337 -l shebang0
$ ls -a
.  ..  .flag.txt
$ cat .flag.txt
CYCTF{w3ll_1_gu3$$_b@sh_1s_e@zy}
```
