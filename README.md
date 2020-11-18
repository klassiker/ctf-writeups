# CTF Writeups

## About this repository

### Structure

You can find writeups for various CTF challenges here. To keep them organized I sorted them by year and event.

Inside each event you can find all writeups that I produced.

Sometimes there are sorted into folders corresponding to the category. This happens only when the writeup is interesting, there were many tasks in that category or just a lot of writeups for that event.

I might change that in the future, I didn't want to create folders for each month because some start and end in differnt months. After publishing the first ones and referencing them on different platforms changing the URL was a no-go, so I sticked with that, at least for the current year to make things consistent.

### Ambition

This can be used as a resource on how to approach certain challenges. Most writeups are limited to a specific task so you will have to search for them yourself.

I will try to create a list of used tools below.

### Indexing

Interesting challenges that you should know and understand can be found in the next section. Those are the ones I liked the most and where I tried to make everything as clear as possible writing my solution down after approaching the task.

### Eventualities

If you have any questions, suggestions or improvements feel free to collaborate using issues or pull requests.

## Things worth reading

### Reverse Engineering

 - [Syskron Security CTF - Key generator](2020/syskron/key-generator.md) Analyzing an unstripped binary to find a hidden secret
 - [Newark Academy CTF - Patches](2020/newark-academy/reverse-engineering/patches.md) Finding a hidden functionality and executing it
 - [Newark Academy CTF - Encoder](2020/newark-academy/reverse-engineering/encoder.md) Reversing assembler with syscalls and reconstructing it in python
 - [Hack.lu CTF - flagdroid](2020/hacklu/flagdroid.md) Analyzing an android APK with apktool and jd by reconstructing dalvik opcodes to java
 - [AppSec IL CTF - GreatSuccess](2020/appsec-il/greatsuccess.md) Analyzing an iOS App Store Package using a React Native jsbundle

### Binary Exploitation

 - [Sunshine CTF - speedrun](2020/sunshine/speedrun/) Collection of 18 different challenges bypassing different security measures
 - [Newark Academy CTF - dROPit](2020/newark-academy/binary-exploitation/dropit.md) Use a stack buffer overflow to inject a ROP-Chain in a binary with no stack protector using a libc leak
 - [Square CTF - Jimi Jam](2020/square/jimi-jam.md) Inject a ROP-Chain in a binary with PIE using an address leak
 - [Sunshine CTF - speedrun-12](2020/sunshine/speedrun/speedrun-12.md) Using an address leak and a format string vulnerability in a binary with PIE
 - [Sunshine CTF - speedrun-14](2020/sunshine/speedrun/speedrun-14.md) Using a ROP-Chain in a statically linked binary without PIE for a syscall to execve

### Web Exploitation

 - [Hack.lu CTF - Confessions](2020/hacklu/confessions.md) Finding hidden data on open GraphQL endpoints
 - [Cyber Security Rumble CTF- Wheels N Whales](2020/cybersecurityrumble/wheels-n-whales.md) Dangerous usage of the default yaml.Loader for remote code execution
 - [AppSec IL CTF - Township Leak](2020/appsec-il/township-leak.md) Bypass access restrictions using prototype pollution on a website using a vulnerable npm package

### Cryptography

 - [Newark Academy CTF - Random Number Generator](2020/newark-academy/random-number-generator.md) Bruteforcing the time-based seed of a PRNG in python to predict the next values
 - [Ledger Donjon CTF - One Time-Based Signature](2020/ledger-donjon/one-time-based-signature.md) Bruteforcing the time-based seed of a math/rand in golang to restore a private key
 - [Ledger Donjon CTF - Secret RNG](2020/ledger-donjon/secret-rng.md) Predicting golang math/rand by observing the output
 - [Square CTF - Hash My Awesome Commands](2020/square/hash-my-awesome-commands.md) Timing attack on a bad implementation of HMAC verification
 - [Affinity CTF Lite - BreakMe](2020/affinity-lite/crypto/breakme.md) Prime factorization of a weak RSA key
 - [Affinity CTF Lite - Collision Course](2020/affinity-lite/crypto/collision-course.md) md5sum collision using HashClash

### Forensics

 - [Syskron Security CTF - HID](2020/syskron/hid.md) Decode the content of a Rubbyer Ducky BadUSB device
 - [Affinity CTF Lite - Classic Forensic](2020/affinity-lite/forensics/classic-forensic.md) Using volatility and dumpchk on a Windows Crash Dump

## Tools

### Reverse & Pwn
 - [radare2](https://github.com/radareorg/radare2), a reverse engineering framework for the command line
 - [PEDA](https://github.com/longld/peda), a python exploit development assistance for gdb
 - [Ghidra](https://github.com/NationalSecurityAgency/ghidra), a reverse engineering framework
 - [pwntools](https://github.com/Gallopsled/pwntools), a CTF framework and exploit development library
 - [apktool](https://github.com/iBotPeaches/Apktool), a tool for reverse engineering Android apk files
 - [jq-gui](https://github.com/java-decompiler/jd-gui), a decompiler for CLASS files inside JARs
 - [jadx](https://github.com/skylot/jadx), another decompiler for JAR/DEX files

### Forensics
 - [volatility](https://github.com/volatilityfoundation/volatility), an advanced memory forensics framework
 - [dumpchk](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/dumpchk), a program that performs a quick analysis of a windows crash dump file
