# Arithmetic

## Task

Ian is exceptionally bad at arthimetic and needs some help on a problem: x + 2718281828 = 42. This should be simple... right?

nc challenges.ctfd.io 30165

File: arithmetic.c

## Solution

I didn't even notice the file. Wow.

```c
int main() {
    setvbuf(stdout, NULL, _IONBF, 0);
    printf("Enter your number:\n");
    int64_t a;
    scanf("%lld", &a);
    if (a < 0) {
        puts("No negative integer inputs!");
        return 1;
    }
    if ((uint32_t) a + (uint32_t) 2718281828 != 42) {
        puts("Not quite!");
        return 1;
    }
    win();
    return 0;
}
```

We need to overflow a 32 bit integer to get 42. Calculation is this: `2**32  - 2718281828 + 42`

```bash
$ nc challenges.ctfd.io 30165
Enter your number:
1576685510
flag: nactf{0verfl0w_1s_c00l_6e3bk1t5}
```
