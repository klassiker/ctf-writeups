# Key generator

## Task

This is our official key generator that we use to derive keys from machine numbers. Our developer put a secret in its code. Can you find it?

File: keygen

Tags: reverse-engineering

## Solution

Let's see what we have here:

```bash
$ file keygen
keygen: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=7d6f8eb010acea04bfdcbeef39b6fa8df52ec9bf, for GNU/Linux 3.2.0, not stripped
```

We analyze this file using radare2:

```bash
$ r2 -AA keygen
[0x000010a0]> afl
0x000010a0    1 46           entry0
0x000010d0    4 41   -> 34   sym.deregister_tm_clones
0x00001100    4 57   -> 51   sym.register_tm_clones
0x00001140    5 65   -> 55   sym.__do_global_dtors_aux
0x00001190    1 9            entry.init0
0x00001000    3 27           sym._init
0x00001500    1 5            sym.__libc_csu_fini
0x000012d8    9 213          sym.genserial
0x00001508    1 13           sym._fini
0x00001490    4 101          sym.__libc_csu_init
0x000013ad    6 212          main
0x00001199    8 165          sym.strrev
0x0000123e    6 154          sym.octal
0x00001030    1 6            sym.imp.toupper
0x00001040    1 6            sym.imp.puts
0x00001050    1 6            sym.imp.strlen
0x00001060    1 6            sym.imp.__stack_chk_fail
0x00001070    1 6            sym.imp.printf
0x00001080    1 6            sym.imp.strcmp
0x00001090    1 6            sym.imp.__isoc99_scanf
```

We see the functions `main`, `sym.genserial`, `sym.strrev` and `sym.octal`.

```nasm
[0x000010a0]> pdf@main
            ; DATA XREF from entry0 @ 0x10c1
┌ 212: int main (int argc, char **argv, char **envp);
│           ; var char *s @ rbp-0x10
│           ; var int64_t canary @ rbp-0x8
│           0x000013ad      55             push rbp
│           0x000013ae      4889e5         mov rbp, rsp
│           0x000013b1      4883ec10       sub rsp, 0x10
│           0x000013b5      64488b042528.  mov rax, qword fs:[0x28]
│           0x000013be      488945f8       mov qword [canary], rax
│           0x000013c2      31c0           xor eax, eax
│           0x000013c4      488d3dbd0e00.  lea rdi, [0x00002288]       ; "/********************************************************************************" ; const char *s
│           0x000013cb      e870fcffff     call sym.imp.puts           ; int puts(const char *s)
│           0x000013d0      488d3d090f00.  lea rdi, str.Copyright__C__BB_Industry_a.s.___All_Rights_Reserved ; 0x22e0 ; "* Copyright (C) BB Industry a.s. - All Rights Reserved" ; const char *s
│           0x000013d7      e864fcffff     call sym.imp.puts           ; int puts(const char *s)
│           0x000013dc      488d3d350f00.  lea rdi, str.Unauthorized_copying_of_this_file__via_any_medium_is_strictly_prohibited ; 0x2318 ; "* Unauthorized copying of this file, via any medium is strictly prohibited" ; const char *s
│           0x000013e3      e858fcffff     call sym.imp.puts           ; int puts(const char *s)
│           0x000013e8      488d3d790f00.  lea rdi, str.Proprietary_and_confidential ; 0x2368 ; "* Proprietary and confidential" ; const char *s
│           0x000013ef      e84cfcffff     call sym.imp.puts           ; int puts(const char *s)
│           0x000013f4      488d3d8d0f00.  lea rdi, str.Written_by_Marie_Tesa__ov____m.tesarova_bb_industry.cz___April_2011 ; 0x2388 ; "* Written by Marie Tesa\u0159ov\u00e1 <m.tesarova@bb-industry.cz>, April 2011" ; const char *s
│           0x000013fb      e840fcffff     call sym.imp.puts           ; int puts(const char *s)
│           0x00001400      488d3dc90f00.  lea rdi, str.               ; 0x23d0 ; "********************************************************************************/\n" ; const char *s
│           0x00001407      e834fcffff     call sym.imp.puts           ; int puts(const char *s)
│           0x0000140c      488d3d151000.  lea rdi, str.Enter_machine_number__e.g._B999999_: ; 0x2428 ; "Enter machine number (e.g. B999999): " ; const char *format
│           0x00001413      b800000000     mov eax, 0
│           0x00001418      e853fcffff     call sym.imp.printf         ; int printf(const char *format)
│           0x0000141d      488d45f0       lea rax, [s]
│           0x00001421      4889c6         mov rsi, rax
│           0x00001424      488d3d231000.  lea rdi, [0x0000244e]       ; "%7s" ; const char *format
│           0x0000142b      b800000000     mov eax, 0
│           0x00001430      e85bfcffff     call sym.imp.__isoc99_scanf ; int scanf(const char *format)
│           0x00001435      488d45f0       lea rax, [s]
│           0x00001439      4889c7         mov rdi, rax                ; const char *s
│           0x0000143c      e80ffcffff     call sym.imp.strlen         ; size_t strlen(const char *s)
│           0x00001441      4883f807       cmp rax, 7
│       ┌─< 0x00001445      7413           je 0x145a
│       │   0x00001447      488d3d0a1000.  lea rdi, str.Invalid_machine_number_format ; 0x2458 ; "Invalid machine number format!" ; const char *s
│       │   0x0000144e      e8edfbffff     call sym.imp.puts           ; int puts(const char *s)
│       │   0x00001453      b801000000     mov eax, 1
│      ┌──< 0x00001458      eb11           jmp 0x146b
│      ││   ; CODE XREF from main @ 0x1445
│      │└─> 0x0000145a      488d45f0       lea rax, [s]
│      │    0x0000145e      4889c7         mov rdi, rax
│      │    0x00001461      e872feffff     call sym.genserial
│      │    0x00001466      b800000000     mov eax, 0
│      │    ; CODE XREF from main @ 0x1458
│      └──> 0x0000146b      488b55f8       mov rdx, qword [canary]
│           0x0000146f      64482b142528.  sub rdx, qword fs:[0x28]
│       ┌─< 0x00001478      7405           je 0x147f
│       │   0x0000147a      e8e1fbffff     call sym.imp.__stack_chk_fail ; void __stack_chk_fail(void)
│       │   ; CODE XREF from main @ 0x1478
│       └─> 0x0000147f      c9             leave
└           0x00001480      c3             ret
```

We can see a call to `int scanf(const char *format)` with `%7s` and a following call to `size_t strlen(const char *s)`. If the length of the scanned string is equal to 7 the function `genserial` is called with the string as a parameter. Let's analyze that function.

```nasm
[0x000010a0]> pdf@sym.genserial
            ; CALL XREF from main @ 0x1461
┌ 213: sym.genserial (char *arg1);
│           ; var char *s1 @ rbp-0x28
│           ; var uint32_t var_14h @ rbp-0x14
│           ; var int64_t var_10h @ rbp-0x10
│           ; var int64_t canary @ rbp-0x8
│           ; arg char *arg1 @ rdi
│           0x000012d8      55             push rbp
│           0x000012d9      4889e5         mov rbp, rsp
│           0x000012dc      4883ec30       sub rsp, 0x30
│           0x000012e0      48897dd8       mov qword [s1], rdi         ; arg1
│           0x000012e4      64488b042528.  mov rax, qword fs:[0x28]
│           0x000012ed      488945f8       mov qword [canary], rax
│           0x000012f1      31c0           xor eax, eax
│           0x000012f3      48b82121616b.  movabs rax, 0x6c61736b612121 ; '!!aksal'
│           0x000012fd      488945f0       mov qword [var_10h], rax
│           0x00001301      488d45f0       lea rax, [var_10h]
│           0x00001305      4889c7         mov rdi, rax                ; char *arg1
│           0x00001308      e88cfeffff     call sym.strrev
│           0x0000130d      4889c2         mov rdx, rax
│           0x00001310      488b45d8       mov rax, qword [s1]
│           0x00001314      4889d6         mov rsi, rdx                ; const char *s2
│           0x00001317      4889c7         mov rdi, rax                ; const char *s1
│           0x0000131a      e861fdffff     call sym.imp.strcmp         ; int strcmp(const char *s1, const char *s2)
│           0x0000131f      85c0           test eax, eax
│       ┌─< 0x00001321      7469           je 0x138c
│       │   0x00001323      488b45d8       mov rax, qword [s1]
│       │   0x00001327      4889c6         mov rsi, rax
│       │   0x0000132a      488d3d370f00.  lea rdi, str.Key_for__s:    ; 0x2268 ; "Key for %s: " ; const char *format
│       │   0x00001331      b800000000     mov eax, 0
│       │   0x00001336      e835fdffff     call sym.imp.printf         ; int printf(const char *format)
│       │   0x0000133b      c745ec060000.  mov dword [var_14h], 6
│      ┌──< 0x00001342      eb34           jmp 0x1378
│      ││   ; CODE XREF from sym.genserial @ 0x137c
│     ┌───> 0x00001344      8b45ec         mov eax, dword [var_14h]
│     ╎││   0x00001347      4863d0         movsxd rdx, eax
│     ╎││   0x0000134a      488b45d8       mov rax, qword [s1]
│     ╎││   0x0000134e      4801d0         add rax, rdx
│     ╎││   0x00001351      0fb600         movzx eax, byte [rax]
│     ╎││   0x00001354      0fbec0         movsx eax, al
│     ╎││   0x00001357      89c7           mov edi, eax                ; int c
│     ╎││   0x00001359      e8d2fcffff     call sym.imp.toupper        ; int toupper(int c)
│     ╎││   0x0000135e      2b45ec         sub eax, dword [var_14h]
│     ╎││   0x00001361      89c6           mov esi, eax
│     ╎││   0x00001363      488d3db60c00.  lea rdi, [0x00002020]       ; "%d" ; const char *format
│     ╎││   0x0000136a      b800000000     mov eax, 0
│     ╎││   0x0000136f      e8fcfcffff     call sym.imp.printf         ; int printf(const char *format)
│     ╎││   0x00001374      836dec01       sub dword [var_14h], 1
│     ╎││   ; CODE XREF from sym.genserial @ 0x1342
│     ╎└──> 0x00001378      837dec00       cmp dword [var_14h], 0
│     └───< 0x0000137c      79c6           jns 0x1344
│       │   0x0000137e      488d3df00e00.  lea rdi, str.DO_NOT_SHARE   ; 0x2275 ; "\nDO NOT SHARE!!!!" ; const char *s
│       │   0x00001385      e8b6fcffff     call sym.imp.puts           ; int puts(const char *s)
│      ┌──< 0x0000138a      eb0a           jmp 0x1396
│      ││   ; CODE XREF from sym.genserial @ 0x1321
│      │└─> 0x0000138c      b800000000     mov eax, 0
│      │    0x00001391      e8a8feffff     call sym.octal
│      │    ; CODE XREF from sym.genserial @ 0x138a
│      └──> 0x00001396      90             nop
│           0x00001397      488b45f8       mov rax, qword [canary]
│           0x0000139b      64482b042528.  sub rax, qword fs:[0x28]
│       ┌─< 0x000013a4      7405           je 0x13ab
│       │   0x000013a6      e8b5fcffff     call sym.imp.__stack_chk_fail ; void __stack_chk_fail(void)
│       │   ; CODE XREF from sym.genserial @ 0x13a4
│       └─> 0x000013ab      c9             leave
└           0x000013ac      c3             ret
```

 - `arg char *arg1 @ rdi` is moved into `s1` `mov qword [s1], rdi`
 - A string `!!aksal` being moved into the `rax` `movabs rax, 0x6c61736b612121`.

`In 64-bit code, movabs can be used to encode the mov instruction with the 64-bit displacement or immediate operand.`

We then see some more moving around of that value and a call to `strrev` with this string. Let's just assume that means `string reverse`.

Then the function `int strcmp(const char *s1, const char *s2)` is called with the probably reversed string and `s1`. If the strings are equal the function `octal` is called. Otherwise the serial will be generated.

We assume that the flag is in the `octal` function. We will reverse this function to find out what it is doing.

Actually we could just execute it and enter `laska!!` as the serial number and see what happens, but where is the fun in that?

```nasm
[0x000010a0]> pdf@sym.octal
            ; CALL XREF from sym.genserial @ 0x1391
┌ 154: sym.octal ();
│           ; var signed int64_t var_214h @ rbp-0x214
│           ; var int64_t var_210h @ rbp-0x210
│           ; var int64_t canary @ rbp-0x8
│           0x0000123e      55             push rbp
│           0x0000123f      4889e5         mov rbp, rsp
│           0x00001242      4881ec200200.  sub rsp, 0x220
│           0x00001249      64488b042528.  mov rax, qword fs:[0x28]
│           0x00001252      488945f8       mov qword [canary], rax
│           0x00001256      31c0           xor eax, eax
│           0x00001258      488d85f0fdff.  lea rax, [var_210h]
│           0x0000125f      488d15fa0d00.  lea rdx, [0x00002060]
│           0x00001266      b941000000     mov ecx, 0x41               ; 'A'
│           0x0000126b      4889c7         mov rdi, rax
│           0x0000126e      4889d6         mov rsi, rdx
│           0x00001271      f348a5         rep movsq qword [rdi], qword ptr [rsi]
│           0x00001274      c785ecfdffff.  mov dword [var_214h], 0
│       ┌─< 0x0000127e      eb29           jmp 0x12a9
│       │   ; CODE XREF from sym.octal @ 0x12b3
│      ┌──> 0x00001280      8b85ecfdffff   mov eax, dword [var_214h]
│      ╎│   0x00001286      4898           cdqe
│      ╎│   0x00001288      8b8485f0fdff.  mov eax, dword [rbp + rax*4 - 0x210]
│      ╎│   0x0000128f      89c6           mov esi, eax
│      ╎│   0x00001291      488d3d880d00.  lea rdi, [0x00002020]       ; "%d" ; const char *format
│      ╎│   0x00001298      b800000000     mov eax, 0
│      ╎│   0x0000129d      e8cefdffff     call sym.imp.printf         ; int printf(const char *format)
│      ╎│   0x000012a2      8385ecfdffff.  add dword [var_214h], 1
│      ╎│   ; CODE XREF from sym.octal @ 0x127e
│      ╎└─> 0x000012a9      81bdecfdffff.  cmp dword [var_214h], 0x81
│      └──< 0x000012b3      7ecb           jle 0x1280
│           0x000012b5      488d3d6c0d00.  lea rdi, str.You_are_not_done_yet ; 0x2028 ; "\nYou are not done yet! \u0ca0\u203f\u0ca0" ; const char *s
│           0x000012bc      e87ffdffff     call sym.imp.puts           ; int puts(const char *s)
│           0x000012c1      90             nop
│           0x000012c2      488b45f8       mov rax, qword [canary]
│           0x000012c6      64482b042528.  sub rax, qword fs:[0x28]
│       ┌─< 0x000012cf      7405           je 0x12d6
│       │   0x000012d1      e88afdffff     call sym.imp.__stack_chk_fail ; void __stack_chk_fail(void)
│       │   ; CODE XREF from sym.octal @ 0x12cf
│       └─> 0x000012d6      c9             leave
└           0x000012d7      c3             ret
```

First:
 - Two addresses are loaded `lea rax, [var_210h]` and `lea rdx, [0x00002060]`
 - `0x41` is moved into `ecx`
 - `rax` is moved into `rdi`, the desination register for string operations
 - `rdx` is moved into `rsi`, the source register for string operations
 - The content of the address `0x00002060` is loaded into `var_210h` `rep movsq qword [rdi], qword ptr [rsi]`

The content is now located between `rbp-0x210` and `rbp-0x8` (size: 0x210 - 0x8 = 520 which is equal to 0x41 * 8, the qword size)

rep: `rep repeats the following string operation ecx times.`

Second:
 - `var signed int64_t var_214h @ rbp-0x214` is set to 0 `mov dword [var_214h], 0`
 - A jump to `cmp dword [var_214h], 0x81` follows - if `var_214h` is less or equal, we jump back
 - `var_214h` is loaded into the `eax` and `cdqe` is called
 - 4 bytes are then loaded from the previous filled region `mov eax, dword [rbp + rax*4 - 0x210]`
 - Those 4 bytes are moved into `esi`, another source register for string operations
 - "%d" is loaded from an address into `rdi`
 - `int printf(const char *format)` is called with these arguments
 - `var_214h` is incremented `add dword [var_214h], 1` and we are back at the `cmp` jump.

This is a simple for-loop which iterates over the bytes and prints them out.

cdqe: `In 64-bit mode, the default operation size is the size of the destination register. Use of the REX.W prefix promotes this instruction (CDQE when promoted) to operate on 64-bit operands. In which case, CDQE copies the sign (bit 31) of the doubleword in the EAX register into the high 32 bits of RAX.`

So, let's look at the address 0x00002060. We want to print 520 bytes as 4 byte pairs from the address, so we hexdump with `px`, add options to only print 130 hexadecimal words `px/130xw`

```nasm
[0x000010a0]> px/130xw 0x00002060
0x00002060  0x00000001 0x00000006 0x00000003 0x00000009  ................
0x00002070  0x00000001 0x00000007 0x00000001 0x00000009  ................
0x00002080  0x00000001 0x00000006 0x00000003 0x00000009  ................
0x00002090  0x00000001 0x00000005 0x00000003 0x00000009  ................
0x000020a0  0x00000001 0x00000006 0x00000002 0x00000009  ................
0x000020b0  0x00000001 0x00000005 0x00000007 0x00000009  ................
0x000020c0  0x00000001 0x00000005 0x00000006 0x00000009  ................
0x000020d0  0x00000001 0x00000000 0x00000003 0x00000009  ................
0x000020e0  0x00000001 0x00000002 0x00000004 0x00000009  ................
0x000020f0  0x00000001 0x00000000 0x00000006 0x00000009  ................
0x00002100  0x00000001 0x00000007 0x00000003 0x00000009  ................
0x00002110  0x00000006 0x00000007 0x00000009 0x00000001  ................
0x00002120  0x00000001 0x00000000 0x00000009 0x00000001  ................
0x00002130  0x00000001 0x00000001 0x00000009 0x00000001  ................
0x00002140  0x00000002 0x00000003 0x00000009 0x00000005  ................
0x00002150  0x00000005 0x00000009 0x00000001 0x00000005  ................
0x00002160  0x00000001 0x00000009 0x00000001 0x00000006  ................
0x00002170  0x00000003 0x00000009 0x00000001 0x00000005  ................
0x00002180  0x00000006 0x00000009 0x00000006 0x00000007  ................
0x00002190  0x00000009 0x00000005 0x00000005 0x00000009  ................
0x000021a0  0x00000001 0x00000006 0x00000003 0x00000009  ................
0x000021b0  0x00000006 0x00000003 0x00000009 0x00000001  ................
0x000021c0  0x00000004 0x00000003 0x00000009 0x00000001  ................
0x000021d0  0x00000002 0x00000005 0x00000009 0x00000001  ................
0x000021e0  0x00000006 0x00000002 0x00000009 0x00000006  ................
0x000021f0  0x00000003 0x00000009 0x00000005 0x00000005  ................
0x00002200  0x00000009 0x00000001 0x00000004 0x00000003  ................
0x00002210  0x00000009 0x00000006 0x00000000 0x00000009  ................
0x00002220  0x00000001 0x00000000 0x00000004 0x00000009  ................
0x00002230  0x00000001 0x00000001 0x00000001 0x00000009  ................
0x00002240  0x00000001 0x00000001 0x00000006 0x00000009  ................
0x00002250  0x00000007 0x00000001 0x00000009 0x00000001  ................
0x00002260  0x00000007 0x00000005                        ........
```

We can now use this data in the language of our choice:

```python
lines = """0x00002060  0x00000001 0x00000006 0x00000003 0x00000009  ................
0x00002070  0x00000001 0x00000007 0x00000001 0x00000009  ................
0x00002080  0x00000001 0x00000006 0x00000003 0x00000009  ................
0x00002090  0x00000001 0x00000005 0x00000003 0x00000009  ................
0x000020a0  0x00000001 0x00000006 0x00000002 0x00000009  ................
0x000020b0  0x00000001 0x00000005 0x00000007 0x00000009  ................
0x000020c0  0x00000001 0x00000005 0x00000006 0x00000009  ................
0x000020d0  0x00000001 0x00000000 0x00000003 0x00000009  ................
0x000020e0  0x00000001 0x00000002 0x00000004 0x00000009  ................
0x000020f0  0x00000001 0x00000000 0x00000006 0x00000009  ................
0x00002100  0x00000001 0x00000007 0x00000003 0x00000009  ................
0x00002110  0x00000006 0x00000007 0x00000009 0x00000001  ................
0x00002120  0x00000001 0x00000000 0x00000009 0x00000001  ................
0x00002130  0x00000001 0x00000001 0x00000009 0x00000001  ................
0x00002140  0x00000002 0x00000003 0x00000009 0x00000005  ................
0x00002150  0x00000005 0x00000009 0x00000001 0x00000005  ................
0x00002160  0x00000001 0x00000009 0x00000001 0x00000006  ................
0x00002170  0x00000003 0x00000009 0x00000001 0x00000005  ................
0x00002180  0x00000006 0x00000009 0x00000006 0x00000007  ................
0x00002190  0x00000009 0x00000005 0x00000005 0x00000009  ................
0x000021a0  0x00000001 0x00000006 0x00000003 0x00000009  ................
0x000021b0  0x00000006 0x00000003 0x00000009 0x00000001  ................
0x000021c0  0x00000004 0x00000003 0x00000009 0x00000001  ................
0x000021d0  0x00000002 0x00000005 0x00000009 0x00000001  ................
0x000021e0  0x00000006 0x00000002 0x00000009 0x00000006  ................
0x000021f0  0x00000003 0x00000009 0x00000005 0x00000005  ................
0x00002200  0x00000009 0x00000001 0x00000004 0x00000003  ................
0x00002210  0x00000009 0x00000006 0x00000000 0x00000009  ................
0x00002220  0x00000001 0x00000000 0x00000004 0x00000009  ................
0x00002230  0x00000001 0x00000001 0x00000001 0x00000009  ................
0x00002240  0x00000001 0x00000001 0x00000006 0x00000009  ................
0x00002250  0x00000007 0x00000001 0x00000009 0x00000001  ................
0x00002260  0x00000007 0x00000005                        ........""".split("\n")

n = [int(y, 16) for x in lines for y in x.split(" ")[2:6] if y != ""]
print(''.join([str(x) for x in n]))
```

We create a flat array of all numbers converted from hex to int. We get:

```bash
$ python solve.py
1639171916391539162915791569103912491069173967911091119123955915191639156967955916396391439125916296395591439609104911191169719175
```

Now we need to find out what to do with this number. The generating function is named `octal`, but this number contains a 9.

It took me a while to figure this out.

If we convert the integers to chars using `chr()` we get this:

```python
['\x01', '\x06', '\x03', '\t', '\x01', '\x07', '\x01', '\t', '\x01', '\x06', '\x03', '\t', '\x01', '\x05', '\x03', '\t', '\x01', '\x06', '\x02', '\t', '\x01', '\x05', '\x07', '\t', '\x01', '\x05', '\x06', '\t', '\x01', '\x00', '\x03', '\t', '\x01', '\x02', '\x04', '\t', '\x01', '\x00', '\x06', '\t', '\x01', '\x07', '\x03', '\t', '\x06', '\x07', '\t', '\x01', '\x01', '\x00', '\t', '\x01', '\x01', '\x01', '\t', '\x01', '\x02', '\x03', '\t', '\x05', '\x05', '\t', '\x01', '\x05', '\x01', '\t', '\x01', '\x06', '\x03', '\t', '\x01', '\x05', '\x06', '\t', '\x06', '\x07', '\t', '\x05', '\x05', '\t', '\x01', '\x06', '\x03', '\t', '\x06', '\x03', '\t', '\x01', '\x04', '\x03', '\t', '\x01', '\x02', '\x05', '\t', '\x01', '\x06', '\x02', '\t', '\x06', '\x03', '\t', '\x05', '\x05', '\t', '\x01', '\x04', '\x03', '\t', '\x06', '\x00', '\t', '\x01', '\x00', '\x04', '\t', '\x01', '\x01', '\x01', '\t', '\x01', '\x01', '\x06', '\t', '\x07', '\x01', '\t', '\x01', '\x07', '\x05']
```

We can see a lot of numbers from 0-7 and some `\t`. Maybe that separates the chars?

```python
>>> p = ''.join([str(x) for x in n]).split("9")
>>> print(''.join([chr(int(x, 8)) for x in p]))
syskronCTF{7HIS-isn7-s3cUr3-c0DIN9}
```
