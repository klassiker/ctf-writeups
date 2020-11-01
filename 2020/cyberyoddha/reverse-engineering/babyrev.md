# babyrev

## Task

We didn’t want to overwhelm you, so let's start with baby rev;

## Solution

```bash
$ file babyrev.out
babyrev.out: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=f58c6c1dbfa854af63d64523f4aab912b1a525e7, for GNU/Linux 3.2.0, not stripped
```

If we load this up in `radare2` we can see a lot of logic in main, so we start `ghidra` to use the decompiler.

```c
undefined8 main(void) {
  char cVar1;
  int iVar2;
  char input [48];
  int counterThree;
  int counterTwo;
  int counter;
  char local_9;

  puts("Hello World");
  puts(
      "Welcome to the Flag checker program. Please enter your flag here, and we will verify whetherit is correct!"
      );
  __isoc99_scanf(&DAT_00102084,input);
  local_9 = '\x01';
  counter = 0;
  while (counter < 0x2b) {
    if ((counter < 0) || (7 < counter)) {
      if ((counter < 9) || (0xf < counter)) {
        if ((counter < 0x11) || (0x18 < counter)) {
          if ((counter < 0x17) || (0x22 < counter)) {
            if ((0x25 < counter) && (counter < 0x2c)) {
              input[counter] = input[counter] + -0xd;
            }
          } else {
            input[counter] = input[counter] + -0xd;
          }
        } else {
          input[counter] = input[counter] + '\a';
        }
      } else {
        input[counter] = input[counter] + -5;
      }
    } else {
      input[counter] = input[counter] + '\x03';
    }
    counter = counter + 1;
  }
  counterTwo = 0;
  while (counterTwo < 0x2b) {
    cVar1 = input[counterTwo];
    if (cVar1 < '\0') {
      cVar1 = cVar1 + '\x03';
    }
    input[counterTwo] = cVar1 >> 2;
    counterTwo = counterTwo + 1;
  }
  counterThree = 0;
  while (counterThree < 0x2b) {
    input[counterThree] = input[counterThree] * '\x03';
    counterThree = counterThree + 1;
  }
  iVar2 = strcmp(input,"3E3?6]QHKTHQQBEETTNKZQ]K]?K<KHH<BQ<KQT<QHNT");
  if (iVar2 != 0) {
    local_9 = '\0';
  }
  if (local_9 == '\0') {
    puts("ACCESS DENIED");
  }
  else {
    puts("ACCESS GRANTED");
  }
  return 0;
}
```

I already renamed some variables.

We can see three for-loops that modify a char array that is scanned in. Let's clean it up:

```c
int main(){
  char input [43] =  {"3E3?6]QHKTHQQBEETTNKZQ]K]?K<KHH<BQ<KQT<QHNT"};

  for (int i = 0; i < 43; i++) {
    if (i > 7) {
      if ((i < 9) || (15 < i)) {
        if ((i < 17) || (24 < i)) {
          if ((i < 23) || (i > 34)) {
            if (i > 37) {
              input[i] = input[i] - 13;
            }
          } else {
            input[i] = input[i] - 13;
          }
        } else {
          input[i] = input[i] + 7;
        }
      } else {
        input[i] = input[i] - 5;
      }
    } else {
      input[i] = input[i] + 3;
    }
  }

  for (int i = 0; i < 43; i++) {
    input[i] = (input[i] >> 2) * 3;
  }

  if (strcmp(input,"3E3?6]QHKTHQQBEETTNKZQ]K]?K<KHH<BQ<KQT<QHNT") != 0) {
    puts("ACCESS DENIED");
  }
  else {
    puts("ACCESS GRANTED");
  }

  return 0;
}
```

In the second for-loop we can see a leftshift 2, so we actually lost the last three bits of the input in the final string. That's a problem but we can fix it.

I copied this code to python to make it easier to work on:

```python
req = "3E3?6]QHKTHQQBEETTNKZQ]K]?K<KHH<BQ<KQT<QHNT"
has = "3E3?6]QHKTHQQBEETTNKZQ]K]?K<KHH<BQ<KQT<QHNT"
input = [ord(x) for x in has];

def change(i):
  do = 0
  if (i > 7):
    if ((i < 9) or (15 < i)):
      if ((i < 17) or (24 < i)):
        if ((i < 23) or (i > 34)):
          if (i > 37) :
            do = -13;
        else:
          do = -13;
      else:
        do = +7;
    else:
      do = -5;
  else:
    do = 3;
  return do

def convert(x, i):
  return ((x + change(i)) >> 2) * 3

for i in range(43):
  input[i] = convert(input[i], i)

poss = [[] for x in range(len(has))]
for i in range(len(has)):
  for x in range(256):
    try:
      if chr(convert(x, i)) == req[i] and chr(x) not in ["[","]","^","`"]:
        poss[i].append(chr(x))
    except Exception as e:
      pass


from itertools import product
for p in product(*poss):
    print(''.join(p))

print(req)
print(''.join([chr(x) for x in input]))
```


Instead of inverting the algorithm I just used bruteforce to find all possible characters for a given position. I could've inverted the computation and add 0-3 to the resulting byte too.

We can now set some values in poss and use the last loop to print all possibilities. Since there are a lot we can slice the array at underscores and find one word at a time. We could use an english dictionary to find words but reading all of them was faster than coding.

After some minute we get: `CYCTF{i_guess_basic_rev_was_too_ez_for_you}`
