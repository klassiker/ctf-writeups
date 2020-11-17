# Classic Forensic

## Task

We need to do some classic forensic stuff on this mem dump, can you help us and check what is important there?

File: mem.dmp.7z

## Solution

Unzip the file:

```bash
$ file MEMORY.DMP
MEMORY.DMP: MS Windows 64bit crash dump, full dump, 262014 pages
```

`volatility` is a great tool for memory forensics: https://github.com/volatilityfoundation/volatility

First we use `imageinfo` to find the correct profile:

```bash
$ volatility -f MEMORY.DMP imageinfo
Volatility Foundation Volatility Framework 2.6.1
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : Win8SP0x64, Win10x64_17134, Win81U1x64, Win10x64_10240_17770, Win2012R2x64_18340, Win10x64_14393, Win10x64, Win2016x64_14393, Win10x64_16299, Win2012R2x64, Win2012x64, Win8SP1x64_18340, Win10x64_10586, Win8SP1x64, Win10x64_15063 (Instantiated with Win10x64_15063)
                     AS Layer1 : SkipDuplicatesAMD64PagedMemory (Kernel AS)
                     AS Layer2 : WindowsCrashDumpSpace64 (Unnamed AS)
                     AS Layer3 : FileAddressSpace (MEMORY.DMP)
                      PAE type : No PAE
                           DTB : 0xa4f000L
                          KDBG : 0xf80002a0e0a0L
          Number of Processors : 2
     Image Type (Service Pack) : 1
                KPCR for CPU 0 : 0xfffff80002a0fd00L
                KPCR for CPU 1 : 0xfffff880009ef000L
             KUSER_SHARED_DATA : 0xfffff78000000000L
           Image date and time : 2020-10-30 21:39:19 UTC+0000
     Image local date and time : 2020-10-30 22:39:19 +0100
```

We now choose a profile and view the process list:

```bash
$ volatility -f MEMORY.DMP --profile=Win2012R2x64_18340 pslist
Volatility Foundation Volatility Framework 2.6.1
Offset(V)          Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit
------------------ -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
0xfffffa800cd20490                           4      0      0 -------- ------      0
0xfffffa800db977f0                         264 12...6      0 -------- ------      0
0xfffffa800ec8df00                         340 12...2      0 -------- ------      0
0xfffffa800eb61f00                         380 42...6      0 -------- ------      0
...
```

Hm, that doesn't look right. Since it's a crash let's try `crashinfo`:

```bash
$ volatility -f MEMORY.DMP --profile=Win2012R2x64_18340 crashinfo
Volatility Foundation Volatility Framework 2.6.1
_DMP_HEADER64:
 Majorversion:         0x0000000f (15)
 Minorversion:         0x00001db1 (7601)
 KdSecondaryVersion    0x00000041
 DirectoryTableBase    0x00a4f000
 PfnDataBase           0xfffffa8000000000
 PsLoadedModuleList    0xfffff80002a62e90
 PsActiveProcessHead   0xfffff80002a44b90
 MachineImageType      0x00008664
 NumberProcessors      0x00000002
 BugCheckCode          0x000000d1
 KdDebuggerDataBlock   0xfffff80002a0e0a0
 ProductType           0x00000001
 SuiteMask             0x00000110
 WriterStatus          0x45474150
 Comment               PAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGEPAGE
 DumpType              Full Dump
 SystemTime            2020-10-30 21:39:19 UTC+0000
 SystemUpTime          0:04:49.497052

Physical Memory Description:
Number of runs: 3
FileOffset    Start Address    Length
00002000      00001000         0009e000
000a0000      00100000         3fde0000
3fe80000      3ff00000         00100000
3ff7f000      3ffff000
```

For other good commands visit: https://code.google.com/archive/p/volatility/wikis/CommandReference21.wiki

For `crashinfo` it mentions: `Information from the crashdump header can be printed using the crashinfo command. You will see information like that of the Microsoft dumpcheck utility.`

It's this tool: https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/dumpchk

You can install the `Debugging Tools for Windows` as explained on that site.

Using `dumpchk.exe` in a Windows VM revealed it's actually Win7 SP1 on x64.

There are lots of other good tools in the tool set Microsoft offers, like `kd` which has a lot of commands to check out.

But let's use `volatility` again:

```bash
$ volatility -f MEMORY.DMP --profile=Win7SP1x64 pslist
Volatility Foundation Volatility Framework 2.6.1
Offset(V)          Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit
------------------ -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
0xfffffa800cd205f0 System                    4      0     86      529 ------      0 2020-10-30 21:34:40 UTC+0000
0xfffffa800db97950 smss.exe                264      4      2       30 ------      0 2020-10-30 21:34:40 UTC+0000
0xfffffa800ec8e060 csrss.exe               340    332      9      423      0      0 2020-10-30 21:34:44 UTC+0000
...
```

That looks more like it!

Looking at almost every possible plugin `volatility` offers, the most interesting part is here:

```bash
$ volatility --profile=Win7SP1x64 -f MEMORY.DMP hashdump
Volatility Foundation Volatility Framework 2.6.1
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
affctf_user:1000:aad3b435b51404eeaad3b435b51404ee:c0042b6eff4211e438835b2d25243133:::
```

So my thought was: how do I crack that password? While researching the method used here I found this: http://moyix.blogspot.com/2008/02/cached-domain-credentials.html

`MD4(MD4(password) + username)`

But way more interesting: It mentions `LSA secrets`. We can dump them too with `volatility`:

```bash
$ volatility --profile=Win7SP1x64 -f MEMORY.DMP lsadump
Volatility Foundation Volatility Framework 2.6.1
DefaultPassword
0x00000000  34 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   4...............
0x00000010  41 00 46 00 46 00 43 00 54 00 46 00 7b 00 66 00   A.F.F.C.T.F.{.f.
0x00000020  30 00 72 00 65 00 6e 00 73 00 69 00 63 00 5f 00   0.r.e.n.s.i.c._.
0x00000030  77 00 33 00 6c 00 6c 00 5f 00 64 00 30 00 6e 00   w.3.l.l._.d.0.n.
0x00000040  33 00 7d 00 00 00 00 00 00 00 00 00 00 00 00 00   3.}.............

DPAPI_SYSTEM
0x00000000  2c 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   ,...............
0x00000010  01 00 00 00 1f 72 e7 50 80 d9 2e 5c 85 d9 6d 97   .....r.P...\..m.
0x00000020  54 66 5f 63 89 10 0d b9 40 24 88 b4 12 c9 73 c1   Tf_c....@$....s.
0x00000030  56 ae ca 01 dc 54 75 12 48 71 42 03 00 00 00 00   V....Tu.HqB.....
```

There we have it!
