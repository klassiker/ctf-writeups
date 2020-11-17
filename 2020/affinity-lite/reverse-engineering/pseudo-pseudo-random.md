# pseudo-pseudo random

## Task

Our hackers have managed to capture the message sent from the enemy's organization computer. It looks like they have their own encryption algorithm but we were also able to get the program responsible for encrypting the messages. Are you able to reverse the algorithm and get the original message?

File: task message

## Solution

First look at the message:

```bash
cat message
The message is: 6351486A40091E7A7A0C0B0249484D43716B305A121B273953
```

Then open the binary in `ghidra` and rename all the things:

```c
int main(int argc,char **argv)
{
  char *argv_one;
  int randomInt;
  size_t strlen_argv_one;
  long in_FS_OFFSET;
  int counterOne;
  int counterTwo;
  int randomValues [8];
  byte dataValues [40];
  long canary;

  canary = *(long *)(in_FS_OFFSET + 0x28);
  argv_one = argv[1];
  if (1 < argc) {
    strlen_argv_one = strlen(argv_one);
    if (strlen_argv_one == 0x19) {
      srand(0x1548);
                    /* fill with "random" */
      counterOne = 0;
      counterTwo = 0;
      while (counterOne < 10) {
        randomInt = rand();
        if ((counterOne & 1U) == 0) {
          randomValues[counterTwo] = randomInt % 0x7f;
          counterTwo = counterTwo + 1;
        }
        counterOne = counterOne + 1;
      }
      counterOne = 0;
      while( true ) {
        strlen_argv_one = strlen(argv_one);
        if (strlen_argv_one <= (ulong)(long)counterOne) break;
        dataValues[counterOne] = argv_one[*(int *)(&DAT_00601080 + (long)counterOne * 4)];
        counterOne = counterOne + 1;
      }
                    /* xor fun */
      counterOne = 0;
      while( true ) {
        strlen_argv_one = strlen(argv_one);
        if (strlen_argv_one <= (ulong)(long)counterOne) break;
        argv_one[counterOne] = (byte)randomValues[counterOne / 5] ^ dataValues[counterOne];
        counterOne = counterOne + 1;
      }
      puts("Here is your encrypted message:");
                    /* printf each char */
      counterOne = 0;
      while( true ) {
        strlen_argv_one = strlen(argv_one);
        if (strlen_argv_one <= (ulong)(long)counterOne) break;
        printf("%02X",(ulong)(uint)(int)argv_one[counterOne]);
        counterOne = counterOne + 1;
      }
                    /* puts newline */
      putchar(10);
      randomInt = 0;
      goto LAB_004008da;
    }
  }
  puts("Please provide the correct input!");
  randomInt = 1;
LAB_004008da:
  if (canary != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return randomInt;
}
```

We have a constant random seed, an array `randomValues` that is filled with the first values of the call to `rand`.

After that the chars of `argv_one` are shuffled using constant values from a data section.

Both values are XORed together and stored in `argv_one`.

The values are then printed as hex. The first argument has to be `0x19` long (25).

We now use the alphabet without `Z` to find out the shuffle order.

Let's set a breakpoint at the last loop, when all variables are initialized. Since the binary is stripped we use `start` and after that `x/128i $rip`.

We locate that position, set a breakpoint and continue.

We find our stack variable offset and print it out.

```nasm
gdb-peda$ start "ABCDEFGHIJKLMNOPQRSTUVWXY"
gdb-peda$ x/128i $rip
...
gdb-peda$ b *0x4008c9
gdb-peda$ c
gdb-peda$ x/8x $rbp-0x60
// random values
0x7fffffffe010: 0x0000004a0000000e      0x0000000500000079
0x7fffffffe020: 0x0000000000000066      0x0000000000000000
// data values
0x7fffffffe030: 0x5145444c56434f58      0x4254484d47534657
0x7fffffffe040: 0x5241594e4b4a5549      0x0000000000000050
```

We can copy them to python now:

```python
import string
import binascii

random = [
0x0000000e,0x0000004a,
0x00000079,0x00000005,
0x00000066,0x00000000,
0x00000000,0x00000000,
]

input = string.ascii_uppercase
data = (0x00000000000000505241594e4b4a55494254484d475346575145444c56434f58).to_bytes(25, "little").decode("ascii")
shuf = []

for c in data:
    shuf.append(input.index(c))

msg = binascii.unhexlify("6351486A40091E7A7A0C0B0249484D43716B305A121B273953")
dec = [x for x in range(0x19)]

for x in range(0x19):
    dec[shuf[x]] = chr(msg[x] ^ random[x // 5])

print(''.join(dec))
```

We need to XOR the `msg` char with the `"random"` values and insert it at the unshuffled position.

Run it:

```bash
$ python solve.py
AFFCTF{1t5_N0t_50_r4nd0m}
```
