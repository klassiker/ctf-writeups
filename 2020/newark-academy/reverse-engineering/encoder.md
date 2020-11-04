# Encoder

## Task

Decode it!

Files: Makefile, encoder, output.bin

## Solution

If you've never done reverse engineering before this challenge might be a bit hard but in my opinion just right. You learn something about syscalls and instructions.

I liked it very much. I have stored these files in the [encoder-files](encoder-files) folder if you want to try it yourself.

Here are some resources:

 - https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/x64-architecture (about registers, their sizes and special access to them)
 - https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/ (about syscalls and their parameters)

Let's start with file file contents:

```bash
$ file encoder
encoder: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=831df3fd066f8dddb47fbbb8a18383c5fc8d3fe4, not stripped
$ file output.bin
output.bin: data
$ cat Makefile
.PHONY: default
default: output.bin

output.bin: flag.txt
        ./encoder <flag.txt >$@
$ xxd output.bin
00000000: 1ece c7be 2ba2 07c8 129e 6236 ae73 76c1  ....+.....b6.sv.
00000010: f294 4b68 c519 bd31 de70 ebbe ad9a fa47  ..Kh...1.p.....G
00000020: 47cd 01                                  G..
```

The task is to reverse engineer what `encoder` does. It reads a flag from `stdin` and writes it to `stdout` in our `output.bin`.

Let's analyze it with `radare2`:

```nasm
[0x00001000]> aaaa
[0x00001000]> afl
0x00001000    8 158          entry0
[0x00001000]> pdf
            ;-- section..text:
            ;-- segment.LOAD1:
            ;-- .text:
            ;-- _start:
            ;-- rip:
┌ 158: entry0 ();
│           ; var int64_t output_char @ rbp-0x10
│           ; var int64_t input_char @ rbp-0x8
│           0x00001000      4889e5         mov rbp, rsp                ; [06] -r-x section size 158 named .text
│           0x00001003      4883ec10       sub rsp, 0x10
│           0x00001007      488d1df20f00.  lea rbx, sym..rodata        ; 0x2000
│           0x0000100e      4d31e4         xor r12, r12
│           ; CODE XREF from entry0 @ 0x1079
│       ┌─> 0x00001011      ba01000000     mov edx, 1
│       ╎   0x00001016      488d75f8       lea rsi, [input_char]
│       ╎   0x0000101a      bf00000000     mov edi, 0
│       ╎   0x0000101f      b800000000     mov eax, 0
│       ╎   0x00001024      0f05           syscall                     
│       ╎   0x00001026      4885c0         test rax, rax
│      ┌──< 0x00001029      7450           je 0x107b                   ; EOF
│      │╎   0x0000102b      480fb645f8     movzx rax, byte [input_char]
│      │╎   0x00001030      480fb6548303   movzx rdx, byte [rbx + rax*4 + 3]
│      │╎   0x00001036      488b0483       mov rax, qword [rbx + rax*4]
│      │╎   0x0000103a      4c89e1         mov rcx, r12
│      │╎   0x0000103d      48d3e0         shl rax, cl
│      │╎   0x00001040      480945f0       or qword [output_char], rax
│      │╎   0x00001044      4901d4         add r12, rdx
│      │╎   0x00001047      4983fc08       cmp r12, 8
│     ┌───< 0x0000104b      722c           jb 0x1079
│     ││╎   0x0000104d      4c89e2         mov rdx, r12
│     ││╎   0x00001050      48c1ea03       shr rdx, 3
│     ││╎   0x00001054      488d75f0       lea rsi, [output_char]
│     ││╎   0x00001058      bf01000000     mov edi, 1
│     ││╎   0x0000105d      b801000000     mov eax, 1
│     ││╎   0x00001062      0f05           syscall                     
│     ││╎   0x00001064      4c89e0         mov rax, r12
│     ││╎   0x00001067      48c1e803       shr rax, 3
│     ││╎   0x0000106b      480fb64405f0   movzx rax, byte [rbp + rax - 0x10]
│     ││╎   0x00001071      488945f0       mov qword [output_char], rax
│     ││╎   0x00001075      4983e407       and r12, 7
│     │││   ; CODE XREF from entry0 @ 0x104b
│     └─└─< 0x00001079      eb96           jmp 0x1011
│      │    ; CODE XREF from entry0 @ 0x1029
│      └──> 0x0000107b      4585e4         test r12d, r12d
│       ┌─< 0x0000107e      7415           je 0x1095
│       │   0x00001080      ba01000000     mov edx, 1
│       │   0x00001085      488d75f0       lea rsi, [output_char]
│       │   0x00001089      bf01000000     mov edi, 1
│       │   0x0000108e      b801000000     mov eax, 1
│       │   0x00001093      0f05           syscall
│       │   ; CODE XREF from entry0 @ 0x107e
│       └─> 0x00001095      31ff           xor edi, edi
│           0x00001097      b83c000000     mov eax, 0x3c               ; '<'
└           0x0000109c      0f05           syscall
```

We can see 4 syscalls (read, write, write, exit). As you can see I already renamed two variables with `afvn output_char var_10h`. Let's add comments with `CCu comment @ addr`:

```nasm
[0x00001000]> pdf
            ;-- section..text:
            ;-- segment.LOAD1:
            ;-- .text:
            ;-- _start:
            ;-- rip:
┌ 158: entry0 ();
│           ; var int64_t output_char @ rbp-0x10
│           ; var int64_t input_char @ rbp-0x8
│           0x00001000      4889e5         mov rbp, rsp                ; [06] -r-x section size 158 named .text
│           0x00001003      4883ec10       sub rsp, 0x10
│           0x00001007      488d1df20f00.  lea rbx, sym..rodata        ; 0x2000
│           0x0000100e      4d31e4         xor r12, r12
│           ; CODE XREF from entry0 @ 0x1079
│       ┌─> 0x00001011      ba01000000     mov edx, 1
│       ╎   0x00001016      488d75f8       lea rsi, [input_char]
│       ╎   0x0000101a      bf00000000     mov edi, 0
│       ╎   0x0000101f      b800000000     mov eax, 0
│       ╎   0x00001024      0f05           syscall                     ; sys_read (0) with size 1
│       ╎   0x00001026      4885c0         test rax, rax
│      ┌──< 0x00001029      7450           je 0x107b
│      │╎   0x0000102b      480fb645f8     movzx rax, byte [input_char]
│      │╎   0x00001030      480fb6548303   movzx rdx, byte [rbx + rax*4 + 3] ; load one byte from rodata (rbx)
│      │╎   0x00001036      488b0483       mov rax, qword [rbx + rax*4] ; load a qword from rodata (rbx)
│      │╎   0x0000103a      4c89e1         mov rcx, r12
│      │╎   0x0000103d      48d3e0         shl rax, cl
│      │╎   0x00001040      480945f0       or qword [output_char], rax
│      │╎   0x00001044      4901d4         add r12, rdx
│      │╎   0x00001047      4983fc08       cmp r12, 8
│     ┌───< 0x0000104b      722c           jb 0x1079                   ; continue loop
│     ││╎   0x0000104d      4c89e2         mov rdx, r12
│     ││╎   0x00001050      48c1ea03       shr rdx, 3
│     ││╎   0x00001054      488d75f0       lea rsi, [output_char]
│     ││╎   0x00001058      bf01000000     mov edi, 1
│     ││╎   0x0000105d      b801000000     mov eax, 1
│     ││╎   0x00001062      0f05           syscall                     ; sys_write (1) with size 'shr rdx, 3'
│     ││╎   0x00001064      4c89e0         mov rax, r12
│     ││╎   0x00001067      48c1e803       shr rax, 3
│     ││╎   0x0000106b      480fb64405f0   movzx rax, byte [rbp + rax - 0x10] ; get last byte from output_char (rbp-0x10)
│     ││╎   0x00001071      488945f0       mov qword [output_char], rax ; load last byte into output_char
│     ││╎   0x00001075      4983e407       and r12, 7
│     │││   ; CODE XREF from entry0 @ 0x104b
│     └─└─< 0x00001079      eb96           jmp 0x1011
│      │    ; CODE XREF from entry0 @ 0x1029
│      └──> 0x0000107b      4585e4         test r12d, r12d             ; check if final char has to be written
│       ┌─< 0x0000107e      7415           je 0x1095
│       │   0x00001080      ba01000000     mov edx, 1
│       │   0x00001085      488d75f0       lea rsi, [output_char]
│       │   0x00001089      bf01000000     mov edi, 1
│       │   0x0000108e      b801000000     mov eax, 1
│       │   0x00001093      0f05           syscall                     ; sys_write (1) with size 1
│       │   ; CODE XREF from entry0 @ 0x107e
│       └─> 0x00001095      31ff           xor edi, edi
│           0x00001097      b83c000000     mov eax, 0x3c               ; '<'
└           0x0000109c      0f05           syscall                     ; sys_exit (0x3c - 60) with code 0
```

So each input byte seems to be modified using data from the `.rodata` section at `0x2000` which is loaded into `rbx`.

Let's extract it to work with that in python. We want to extract 128 times 8 bytes from offset `0x2000`.

Why 128 times? I just used `x/128xg 0x2000` and incremented the amount until I reached junk data. It's also a power of two and just two chars above printable ascii range.

```bash
$ dd if=encoder.bin of=hex.bin ibs=8 count=128 skip=1024
```

Now that we have the `.rodata` section extracted we can read it with python and reconstruct the code. I added the corresponding instruction as a comment on the right. There is some extra shifting and ANDing going on, that's just because python doesn't limits variables like registers do:

**Note:** there might be bugs in this code, I used a refactored version later on where I removed them, this is just an example how to reconstruct.

```python
f = open("hex.bin", "rb")
hex_values = f.read()

def from_bytes(v):
    return int.from_bytes(v, "little")

flag = input() + "\n"
r12 = 0
output_char = 0
# 0x1001
for input_char in flag:
    rax = ord(input_char)                             # movzx rax, byte [input_char]
    rdx = hex_values[rax * 4 + 3]                     # movzx rdx, byte [rbx + rax*4 + 3]
    rax = from_bytes(hex_values[rax * 4:rax * 4 + 8]) # mov rax, qword [rbx + rax*4]
    rcx = r12                                         # mov rcx, r12
    rax = rax << (rcx & 0xff)                         # shl rax, cl
    output_char = output_char | rax                   # or qword [output_char], rax
    r12 = r12 + rdx                                   # add r12, rdx
    if r12 < 8:                                       # cmp r12, 8
        continue                                      # jb 0x1079 -> jmp 0x1011
    rdx = r12                                         # mov rdx, r12
    rdx = rdx >> 3                                    # shr rdx, 3
    print(hex(output_char & 0xff))                    # lea rsi, [output_char] -> syscall
    rax = r12                                         # mov rax, r12
    rax = rax >> 3                                    # shr rax, 3
    rax = output_char >> 8 & 0xff                     # movzx rax, byte [rbp + rax - 0x10]
    output_char = rax                                 # mov qword [output_char], rax
    r12 = r12 & 7                                     # and r12, 7
    # jmp 0x1011

if r12 & 0xffff != 0:                                 # test r12d, r12d -> je 0x1095
    print(hex(output_char & 0xff))                    # lea rsi, [output_char] -> syscall

print()

# 0x1095
exit()                                                # syscall
```

We can verify that this works by encoding the flag prefix or comparing it to the encoder binary:

```bash
 $ xxd output.bin
00000000: 1ece c7be 2ba2 07c8 129e 6236 ae73 76c1  ....+.....b6.sv.
00000010: f294 4b68 c519 bd31 de70 ebbe ad9a fa47  ..Kh...1.p.....G
00000020: 47cd 01
$ echo 'nactf' | ./encoder | xxd
00000000: 1ece c772                                ...r
$ echo 'nactf' | python hex.py
0x1e
0xce
0xc7
0x72
```

Cool. Let's refactor it:

```python
f = open("hex.bin", "rb")
hex_values = f.read()

def from_bytes(v):
    return int.from_bytes(v, "little")

def ph(x):
    print(hex(x))

flag = input() + "\n"
r12 = 0
output_char = 0

for input_char in flag:
    rax = ord(input_char)
    rdx = hex_values[rax * 4 + 3]
    rax = from_bytes(hex_values[rax * 4:rax * 4 + 8])
    rax = rax << (r12 & 0xff)
    output_char = output_char | rax
    r12 += rdx
    if r12 < 8:
        continue
    ph(output_char & 0xff)
    output_char = output_char >> 8 & 0xff
    r12 = r12 & 7

if r12 & 0xffff != 0:
    ph(output_char & 0xff)
```

Now we have to understand what's going on. We actually only use the last 2 bytes of the every 8 byte entry in the `.rodata` section. The first byte is added to `r12`, some kind of counter variable. The second byte is shifted by that counter and then ORed with the value of `output_char`. That's not good, since `OR` is not reversible. There are multiple solutions when reversing this.

If the counter is above or equal to 8 the current `output_char` is printed, we then take the second byte from the left and store that in `output_char`.  `r12` is then limited to 3 bits.

Otherwise we continue with our `r12` and `output_char` values. That makes things even worse because we now have to OR multiple characters together until we print it. If it's not printed we can't see if our current possibility would result in the correct output. We also have to carry the `r12` variable around.

It took me a while and some debugging until I got the algorithm right. We thankfully only have to OR two possible characters at most, that makes the coding a lot easier. This was observed by debugging the binary with different inputs.

This is actually not needed since our reversing function is recursive, but to keep things simple I first made a nested loop to later reduce the complexity of the code and increase the readability. Finding the right parameters for different branches of the recursive call can be hard to debug when you are trying to simulate what an assembly instruction really does.

What I did was this: write a recursive function that tries to add a character and see if the calculated value matches the one in `output.bin` at that position

After finishing that I had to do cleanup work because my code calculated every possible input, but the result was a mess. Arrays in arrays in arrays. I couldn't read it. After an hour of tracking the executions I managed to get it right, it was rather simple. I just had to join each calculated character from the recursive with the current char of the loop.

 Here is the code:

 ```python
 import string

 f = open("hex.bin", "rb")
 hex_values = f.read()
 f.close()

 f = open("output.bin", "rb")
 flag_values = f.read()
 f.close()

 def from_bytes(v):
     return int.from_bytes(v, "little")

 def get_char(rax):
     rdx = hex_values[rax * 4 + 3]
     rax = from_bytes(hex_values[rax * 4:rax * 4 + 8])
     rax = rax << (r12 & 0xff)
     return rax, rdx

 def do_next(offset, r12, output_char):
     if offset + 1 >= len(flag_values):
         return output_char == flag_values[-1], []

     found = False
     poss = []
     search = flag_values[offset]
     for charOne in string.printable:
         rax, rdx = get_char(ord(charOne))
         rax = rax << (r12 & 0xff)
         test_char = output_char | rax
         test_r12 = r12 + rdx
         f, c = None, None

         if test_r12 < 8:
             f, c = do_next(offset, test_r12, test_char)
         else:
             if test_char & 0xff == search and (offset < 33 or output_char <= 0xff):
                 f, c = do_next(offset + 1, test_r12 & 7, test_char >> 8 & 0xff)
         if f:
             found = True
             if len(c) > 0:
                 for p in c:
                     poss.append(charOne + p)
             else:
                 poss.append(charOne)

     return found, poss

 r12 = 0
 output_char = 0
 success, possibilities = do_next(0, r12, output_char)

 for poss in possibilities:
     print(poss.strip())
```

This is the result:

```bash
$ time python hex.py
nactf{4ss3mbly_huffm4n_c0d1ng_1s_fun_6JkDGFG3qTAYVe8h}

real    0m0,460s
user    0m0,350s
sys     0m0,010s
```

Bruteforce would also be a solution but where is the fun in that? This also kind of forcing it by checking each character for each position.

The flag references `Huffman Coding`: https://en.wikipedia.org/wiki/Huffman_coding

`Huffman code is a particular type of optimal prefix code that is commonly used for lossless data compression`

This flag was very important to me because I realized I still had bugs in my code. In the version above they are fixed but before I did that there were multiple output lines, multiple flags and some with garbage at the end. Without that information I would have accepted them as junk, not realizing they were there because of bugs.

The exit condition has to end 1 character before the length of the binary values because the last char is printed in another part of the binary, the second `sys_write`.

The `output_char` for the second last byte (33) must also be below or equal to 255 because of that.

There is probably another way to solve this challenge: reconstructing the huffman tree from the `.rodata` section and decode it the 'right' way. In order to solve this challenge that way you would have to known this upfront or recognize it in the code. I didn't try it out to see if it's possible, so if you didn't solve this, this might be your challenge.
