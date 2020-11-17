# Char Wrap

## Task

File: charwrap

## Solution

We get this file:

```bash
$ file charwrap
charwrap: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=c4dbdaa0b6f21111def5fa448746701f32c2d7ef, for GNU/Linux 3.2.0, not stripped
```

Analyzing it with `radare2` we see a lot of strings in `main`. Let's grep for the flag. We use `-C` for context:

```nasm
[0x00001040]> pdf@main | grep 'AFFCTF{' -C 6
│           0x00001ea2      c685a2f8ffff.  mov byte [var_75he], 0
│           0x00001ea9      48b84f757666.  movabs rax, 0x6e4171676676754f ; 'OuvfgqAn'
│           0x00001eb3      48898571f8ff.  mov qword [var_78fh], rax
│           0x00001eba      c78579f8ffff.  mov dword [var_787h], 0x4c667b6a ; 'j{fL'
│           0x00001ec4      66c7857df8ff.  mov word [var_783h], 0x724f ; 'Or'
│           0x00001ecd      c6857ff8ffff.  mov byte [var_781h], 0
│           0x00001ed4      48b841464643.  movabs rax, 0x797b465443464641 ; 'AFFCTF{y'
│           0x00001ede      48ba6f755f66.  movabs rdx, 0x646e756f665f756f ; 'ou_found'
│           0x00001ee8      48898550f8ff.  mov qword [var_7b0h], rax
│           0x00001eef      48899558f8ff.  mov qword [var_7a8h], rdx
│           0x00001ef6      48b85f736f6d.  movabs rax, 0x696874656d6f735f ; '_somethi'
│           0x00001f00      48898560f8ff.  mov qword [var_7a0h], rax
│           0x00001f07      c78568f8ffff.  mov dword [var_798h], 0x7d21676e ; 'ng!}'
```

Just put it together.

You could also do a dynamic analysis with `gdb` and print the contents in the stack as strings.
