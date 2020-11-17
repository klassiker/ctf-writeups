# Collision Course

## Task

Hello there,

I heard you are spreading rumors that hashing has some drawbacks. I don't believe it and I prove it! Go to http://web6.affinityctf.com/ and create 2 DIFFERENT files with the same md5 hash. Additionally, the files have to contain the phrase: "AFFCTF". File size limit is is 100000b.

Regards, Mr. Your-colleague-who-is-always-right

## Solution

On the website we can upload two files and send them to the server. It checks if they contain `AFFCTF` and that they are different. It then checks the md5 hash, which have to be equal. In the source code we see references to `upload.php`.

It's probably this function: https://www.php.net/manual/en/function.md5-file.php

Trying it locally it's the md5sum of the file.

We now need a tool to calculate an md5sum collision. In this list of ctf-tools you can find a link to `fastcoll`: https://github.com/zardus/ctf-tools

It's actually `HashClash`: https://www.win.tue.nl/hashclash/

The source code is here: https://github.com/cr-marcstevens/hashclash

Now we use this tool to create a collision containing `AFFCTF` in files `first` and `second`:

```bash
$ ./md5_fastcoll -p <(echo "AFFCTF") -o first second
MD5 collision generator v1.5
by Marc Stevens (http://www.win.tue.nl/hashclash/)

Using output filenames: "first" and "second"
Using prefixfile: "/dev/fd/63"
Using initial value: 3914dc0610106b5376845f8b238e27d2

Generating first block: .......................
Generating second block: S01..
Running time: 5.6755 s
$ md5sum first second
7f232126cc679560653c43f77dfe6952  first
7f232126cc679560653c43f77dfe6952  second
```

Uploading them to the service:

```
Checking, please wait...
String found in the first file
String found in the second file
Checking if files are different...
Files are different
Checking if files are MD5 Hash is the same for both files...
MD5Hashes are the same. You were right. The flag is: AFFCTF{One_Way_Or_Another}
```
