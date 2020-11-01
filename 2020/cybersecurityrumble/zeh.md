# Zeh

## Task

For the CSR we finally created a deutsche Programmiersprache!

nc chal.cybersecurityrumble.de 65123

File: haupt.c

## Solution

We get a `main.c` with all keywords and functions defined as german words. Replace all of them with their original value and we get:

```c
#include <stdio.h>
#include <stdlib.h>
#include "fahne.h"

void main(void) {
    int i = rand();
    int k = 13;
    int e;
    int *p  = &i;

    printf("%d\n", i);
    fflush(stdout);
    scanf("%d %d", &k, &e);

    for(int i = 7;i--;) {
      k = (*p) >> (k % 3);
    }

    k = k ^ e;

    if(k == 53225)
        puts(Fahne);
    else
        puts("War wohl nichts!");
}
```

The easiest way to solve this is by setting `k` to a constant value, like 2 and to rewrite this to scan in the random number from the service, process it in the loop and xor it with `53225`.

```c
#include <stdio.h>
#include <stdlib.h>

void main(void) {
    int i = 1804289383;
    scanf("%d", &i);

    int k = 2;
    int e;
    int *p  = &i;

    for(int i = 7;i--;) {
      k = (*p) >> (k % 3);
    }

    e = k ^ 53225;
    printf("%d\n", e);
}
```

The flag is: `CSR{RUECKWARTSINGENEUREN}`
