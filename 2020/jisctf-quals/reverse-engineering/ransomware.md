# Ransomware

## Task

File: Ransomware.zip

## Solution

We have `flag.enc`:

```
703230904e174e26c358bab99598029beb45510803f878d56109cee328b0a2237feb4fcf668cebbc821ff79a9491799b186c4bea62c3cf4f73385267af85374641c0f852703230907032309017e0ea0bb9ac54a1bf515fdabfb6b5c76bed23455e018c743dbd673dfedbf3b098c1a9748d883fc86ff6280754f7ec17510c2ad8fe1d346483e7f69b4c0a60b77032309070323090e9f0da56136d59e6191edb34643309a4b3bc5ffdcdec697aa990545926aca949986c87b924b729f68c8728803a4e21e30e05cf6ed9badb1a829d91cba578adf2703230907032309083077d5bf40f4fd308fc1eb93c87081ae272d6b222a4172e7e258ee5584994d25c22c224ec4789f5cc4c5abcfad1db0445228c53480f6fdecebcc5407dee7c1c7032309070323090c8fb728bed9d756ebb3417e54b598c3785f14157e23bb141e6d10f2c4e851dd5036aa541849c9e97f0fe057b387df683a28c62b9ee97c1e8985914bfb6ec24e2703230907032309093bf4523662ecfea34cb0377210384f91838b449dfd56979a24b2c7904dd1325dde3d1de045985e7e75c02a2506299a96d4e3d16e7331427425af0223c72358c7032309070323090fb6f7bb42da48e274872b8e28fadea4bb8db0e25f858ec7f95a7682dbd4520119f81674f9cb8737034e751a62b321b84d394638ab29dd1c427cf10c1077cba587032309070323090477d78623356e687aec90e173da11db28235442efc40ee35e23ee7af7410b81fae18912458dd96a269a3c857f4915f89de7d164a3d326d8380db44685c7164e570323090703230902d47c7bc766a988367907939450405507227cc9d272120f3a529ddbff61f7a7dcdb7139b29dea09307c3225f043555a4d3447637ef481cfa5a734da900eea58070323090703230903a03d4db2311a18ed5bfa246b1117961f043a5204c8f92e2ce9882605adae8300dad339acfa38b1f6c7c862fdd10147c4aedb9e54889b608c6ccdca3caeefe0b7032309070323090f8ea666ab4890113372c776e2e01180a2b2286054e63ce3884d2b554d0d565e872225f6e766c01ca8236f6f62110dfcc1a909219d089893426f98d2bed9bbdeb70323090703230908daa8b02883790fb38b484777cda22e384c50400f21dc9188f0bcd8244f8aaa01a31b58bc4b7354aa78623a2b9aa9a4f1bb2ec42cc74dd1c42554fdede7c50ef703230907032309003af5a4e9d1b321c2a83bd345a8cccbb2a10282967c9e96c8a4d170a95b1ddae4a187fbdd85157ca0d2eb2e409fc05f84004b86e2033e55135b428843a988a3c703230907032309027fa1dff4a39502ede87dad98a304d34c136fe2abc54c08233c5834ab2a35928a669ac59eb7ae416ab70cb036a1d7ad0486782b9d7761f8457d09f87d85166e07032309070323090969291c04cccd97065722a16eb9c96f12bd8562af2662dd2dfefd03fffd26170a385a2f976254cfdf8686f9139c0893ebb0a3b7b39823fc6299d37a33f2735107032309070323090f9590c29da1b7609c00d144fe34f4c49a737342f336f5cbe574430dffd47295ed84a380c95163eeb401ff2470fc03539c75f8afe5fe5c206117c94315266ff5d703230907032309013abc8956e57ce6be9a2b03184c5619a95022587d3e4c2eb194f1164b0ecbf3352fdcbd91ab0361205c4301d91453f22434d00b6505a536950b12e4819c3ce7d70323090
```

And `Ransomware`:

```bash
$ file Ransomeware
Ransomeware: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=eee1a1e574a7b7b312c9a2c50a2855d4092e17a3, not stripped
```

Let's open it up with `ghidra` and rename all the things:

This is `main`:

```c

/* WARNING: Unknown calling convention yet parameter storage is locked */

int main(void)

{
  byte bVar1;
  int fd;
  int iVar2;
  ssize_t bytesRead;
  size_t flagStrlen;
  time_t epochtime;
  size_t randomStrlen;
  char *random;
  long in_FS_OFFSET;
  int counterTwoReverse;
  int counter;
  int counterTwo;
  uchar charZero;
  uchar charOne;
  uchar flagData [48];
  undefined local_1a8 [64];
  byte sha512_hash [64];
  char sha512_store [128];
  char encData [136];
  long canary;

  canary = *(long *)(in_FS_OFFSET + 0x28);
  bzero(flagData,0x23);
  fd = open("./flag.txt",0);
  if (fd == -1) {
    perror("Error opening the file\n");
  }
  do {
    bytesRead = read(fd,flagData,0x23);
  } while (0 < (int)bytesRead);
  flagStrlen = strlen((char *)flagData);
  close(fd);
  epochtime = time((time_t *)0x0);
  srand((uint)epochtime);
  bzero(local_1a8,0x40);
  fd = open("./out.txt",0x41);
  if (fd == -1) {
    perror("Error opening the file\n");
  }
  counter = 2;
  while ((ulong)(long)counter <= flagStrlen) {
    bzero(&charZero,3);
    bzero(sha512_hash,0x40);
    bzero(sha512_store,0x80);
    bzero(encData,0x80);
    charZero = flagData[counter + -2];
    charOne = flagData[counter + -1];
    rand();
    rand();
    SHA512(&charZero,2,sha512_hash);
    counterTwo = 0;
    while (counterTwo < 0x40) {
      bVar1 = sha512_hash[counterTwo];
      randomStrlen = strlen(sha512_store);
      sprintf(sha512_store + randomStrlen,"%02x",(ulong)bVar1);
      counterTwo = counterTwo + 1;
    }
    randomStrlen = strlen(sha512_store);
    iVar2 = (int)randomStrlen + -1;
    counterTwo = 0;
    counterTwoReverse = iVar2;
    while (counterTwo <= iVar2) {
      if (sha512_store[counterTwoReverse] != ' ') {
        encData[counterTwo] = sha512_store[counterTwoReverse];
        counterTwoReverse = counterTwoReverse + -1;
      }
      counterTwo = counterTwo + 1;
    }
    random = genRandom();
    randomStrlen = strlen(random);
    write(fd,random,randomStrlen);
    write(fd,encData,0x80);
    random = genRandom();
    randomStrlen = strlen(random);
    write(fd,random,randomStrlen);
    counter = counter + 2;
  }
  if (canary != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```

So a flag is read from `./flag.txt` and stored in `./out.txt` (our `flag.enc`). There is some PRNG seeding with time that is never used.

There is a loop that iterates over two chars at a time and calculates the `sha512` hash. The second loop then reverses this hash.

After that a value from `genRandom` is written, then the hash, then another value from `genRandom`.

The value returned from `genRandom` is 8 bytes long. So we start of by extracting the real data.

Then we just need to find a two character combination with the correct hash:

```python
with open("flag.enc", "r") as f:
    d = f.read()

random_size = 8
data_size = 0x80
block_size = random_size*2 + data_size

import string
from hashlib import sha512

out = ""
for x in range(0, len(d), block_size):
    v = d[x+random_size:x+data_size+random_size][::-1]
    for c1 in string.printable:
        for c2 in string.printable:
            if sha512((c1+c2).encode("ascii")).hexdigest() == v:
                out += c1+c2
print(out)
```

Running it:

```bash
$ python rev.py
JISCTF{R3V3Rs1Ng_W1Th_3Nkr3Pt10N}
```
