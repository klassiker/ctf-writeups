# flagdroid

## Task

Categories: rev

Difficulty: easy

This app won't let me in without a secret message. Can you do me a favor and find out what it is?

File: fragdroid.zip

## Solution

We use `apktool` to unpack the apk from the zip:

```bash
$ apktool d --no-src flagdroid.apk
$ cd flagdroid
```

We then convert the `classes.dex` with `dex2jar` to a jar to open it with `jd`:

```bash
$ d2j-dex2jar.sh -d classes.dex
```

We look at the `lu.hack.Flagdroid.MainActivity.class`:

 - a libary is loaded `native-lib`
 - the `onCreate` method matches the text of an `EditText` to a flag pattern
 - if a match is found the flag content is split on underscores
 - if there are 4 parts each part is checked with a `checkSplit[1-4]` function

The challenge is now to reverse engineer each flag part according to the check function.

### Part 1

```java
private boolean checkSplit1(String paramString) {
  byte[] arrayOfByte = Base64.decode(getResources().getString(2131492894), 0);
  try {
    return (new String(arrayOfByte, "UTF-8")).equals(paramString);
  } catch (UnsupportedEncodingException unsupportedEncodingException) {
    return false;
  }
}
```

A string resource is loaded and base64 decoded. This is the flag part.

We take a look at: https://developer.android.com/guide/topics/resources/providing-resources

The table lists the directory `values` and `strings.xml` for string values.

We therefore open `res/values/strings.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
  ...
  <string name="encoded">dEg0VA==</string>
  ...
</resources>
```

First part: tH4T

### Part 2

This one failed to decompile, so we need to reconstruct the opcodes to java. We could also do this with the bytecode from `jd`, but I prefer the `smali` files apktool produces when also decompiling the sources.

```bash
$ apktool d --no-res -o sources flagdroid.apk
$ cd sources/smali/lu/hack/Flagdroid/
```

We can now take a look at the `MainActivity.smali`:

```smali
.method private checkSplit2(Ljava/lang/String;)Z
    .locals 7

    const-string v0, "\u001fTT:\u001f5\u00f1HG"

    const/4 v1, 0x0

    .line 131
    :try_start_0
    invoke-virtual {p1}, Ljava/lang/String;->toCharArray()[C

    move-result-object p1

    const-string v2, "hack.lu20"

    const-string v3, "UTF-8"

    .line 132
    invoke-virtual {v2, v3}, Ljava/lang/String;->getBytes(Ljava/lang/String;)[B

    move-result-object v2

    .line 134
    array-length v3, p1

    const/16 v4, 0x9

    if-eq v3, v4, :cond_0

    return v1

    :cond_0
    const/4 v3, 0x0

    :goto_0
    if-ge v3, v4, :cond_1

    .line 140
    aget-char v5, p1, v3

    add-int/2addr v5, v3

    int-to-char v5, v5

    aput-char v5, p1, v3

    .line 141
    aget-char v5, p1, v3

    aget-byte v6, v2, v3

    xor-int/2addr v5, v6

    int-to-char v5, v5

    aput-char v5, p1, v3

    add-int/lit8 v3, v3, 0x1

    goto :goto_0

    .line 143
    :cond_1
    invoke-static {p1}, Ljava/lang/String;->valueOf([C)Ljava/lang/String;

    move-result-object p1

    invoke-virtual {p1, v0}, Ljava/lang/String;->equals(Ljava/lang/Object;)Z

    move-result p1
    :try_end_0
    .catch Ljava/io/UnsupportedEncodingException; {:try_start_0 .. :try_end_0} :catch_0

    return p1

    :catch_0
    return v1
.end method
```

This might be helpful: http://pallergabor.uw.hu/androidblog/dalvik_opcodes.html

After some time we get (I left out the try-catch to keep it simple, it is also just a return of `v1b`):

```java
private static boolean checkSplit2(String p1s) throws UnsupportedEncodingException {
   // const-string v0, "\u001fTT:\u001f5\u00f1HG"
   String v0s = "\u001fTT:\u001f5\u00f1HG";

   // const/4 v1, 0x0
   boolean v1b = false;

   // invoke-virtual {p1}, Ljava/lang/String;->toCharArray()[C
   // move-result-object p1
   char[] p1ca = p1s.toCharArray();

   // const-string v2, "hack.lu20"
   String v2s = "hack.lu20";

   // const-string v3, "UTF-8"
   String v3s = "UTF-8";

   // invoke-virtual {v2, v3}, Ljava/lang/String;->getBytes(Ljava/lang/String;)[B
   // move-result-object v2
   byte[] v2ba = v2s.getBytes(v3s);

   // array-length v3, p1
   int v3i = p1ca.length;

   // const/16 v4, 0x9
   int v4i = 0x9;

   // if-eq v3, v4, :cond_0
   if (v3i == v4i) {
       // :cond_0

       // const/4 v3, 0x0
       v3i = 0;

       // if-ge v3, v4, :cond_1
       while (v3i < v4i) {
           // aget-char v5, p1, v3
           int v5i = p1ca[v3i];

           // add-int/2addr v5, v3
           v5i += v3i;

           // int-to-char v5, v5
           char v5c = (char) v5i;

           // aput-char v5, p1, v3
           p1ca[v3i] = v5c;

           // aget-char v5, p1, v3
           v5i = p1ca[v3i];

           // aget-byte v6, v2, v3
           byte v6b = v2ba[v3i];

           // xor-int/2addr v5, v6
           v5i = v5i ^ v6b;

           // int-to-char v5, v5
           v5c = (char) v5i;

           // aput-char v5, p1, v3
           p1ca[v3i] = v5c;

           // add-int/lit8 v3, v3, 0x1
           v3i = v3i + 1;
       }

       // :cond_1

       // invoke-static {p1}, Ljava/lang/String;->valueOf([C)Ljava/lang/String;
       // move-result-object p1
       p1s = String.valueOf(p1ca);

       // invoke-virtual {p1, v0}, Ljava/lang/String;->equals(Ljava/lang/Object;)Z
       // move-result p1
       boolean p1b = p1s.equals(v0s);

       return p1b;
   } else {
       return v1b;
   }
}
```

That looks complicated. Let's break it down:

```java
private static boolean checkSplit2(String paramString) {
        String expected = "\u001fTT:\u001f5\u00f1HG";
        boolean equals = false;

        char[] charArray = paramString.toCharArray();
        byte[] byteArray = "hack.lu20".getBytes(StandardCharsets.UTF_8);

        if (charArray.length == 9) {
            for (int i = 0; i < charArray.length; i++) {
                charArray[i] = (char) ((charArray[i] + i) ^ byteArray[i]);
            }

            equals = String.valueOf(charArray).equals(expected);
        }

        return equals;
    }
```

Now we need a function to reverse this:

```java
private static void reverseSplit2() {
    String res = "\u001fTT:\u001f5\u00f1HG";
    byte[] xor = "hack.lu20".getBytes(StandardCharsets.UTF_8);
    char[] inp = res.toCharArray();

    for (int i = 0; i < 9; i++) {
        System.out.print((char) ((inp[i] ^ xor[i]) - i));
    }

    System.out.println();
}
```

Second part: w45N-T~so

### Part 3

```java
private boolean checkSplit3(String paramString) {
  paramString = paramString.toLowerCase();
  return (paramString.length() != 8) ? false : (!paramString.substring(0, 4).equals("h4rd") ? false : md5(paramString).equals("6d90ca30c5de200fe9f671abb2dd704e"));
}
```

What we know:
 - length has to be 8.
 - first 4 characters ar `h4rd`
 - md5(string) is 6d90ca30c5de200fe9f671abb2dd704e

 We now write a script that bruteforces the last 4 characters:

 ```python
from hashlib import md5

chars = [x.encode("utf-8") for x in "abcdefghijklmnopqrstuvwxyz1234567890-~!?"]
solve = b'h4rd'
search = "6d90ca30c5de200fe9f671abb2dd704e"

def addChar(s, length=8):
  if len(s) < length:
    for char in chars:
      addChar(s + char)
  else:
    hexdigest = md5(s).hexdigest()
    if hexdigest == search:
      print(s.decode("utf-8"))

addChar(solve)
 ```

You could also use the full ascii range or more special characters, I assumed this chars from the other flag parts to speed up the process. Also no uppercase is needed because the paramString is converted to lowercase before hashing.

Third part: h4rd~huh

### Part 4

```java
private boolean checkSplit4(String paramString) {
  return paramString.equals(stringFromJNI());
}

public native String stringFromJNI();
```

We need to look at the `native-lib` now. It is located under `lib/${arch}/libnative-lib.so`:

```nasm
$ r2 -AA x86_64/libnative-lib.so
[0x000005b0]> afl
0x000005b0    1 12           entry0
0x00000610    1 57           sym.Java_lu_hack_Flagdroid_MainActivity_stringFromJNI
0x000005a0    1 6            sym.imp.malloc
0x00000590    1 6            sym.imp.__cxa_atexit
0x00000580    1 6            sym.imp.__cxa_finalize
0x000005d0    2 21   -> 6    entry.fini0
[0x000005b0]> pdf @ sym.Java_lu_hack_Flagdroid_MainActivity_stringFromJNI
┌ 57: sym.Java_lu_hack_Flagdroid_MainActivity_stringFromJNI (int64_t arg1);
│           ; arg int64_t arg1 @ rdi
│           0x00000610      53             push rbx
│           0x00000611      4889fb         mov rbx, rdi                ; arg1
│           0x00000614      bf0d000000     mov edi, 0xd                ; size_t size
│           0x00000619      e882ffffff     call sym.imp.malloc         ;  void *malloc(size_t size)
│           0x0000061e      c6400874       mov byte [rax + 8], 0x74    ; 't'
│                                                                      ; [0x74:1]=0
│           0x00000622      c740093f3829.  mov dword [rax + 9], 0x29383f ; '?8)'
│                                                                      ; [0x29383f:4]=-1
│           0x00000629      48b930727e77.  movabs rcx, 0x312d5334777e7230 ; '0r~w4S-1'
│           0x00000633      488908         mov qword [rax], rcx
│           0x00000636      488b0b         mov rcx, qword [rbx]
│           0x00000639      488b89380500.  mov rcx, qword [rcx + 0x538]
│           0x00000640      4889df         mov rdi, rbx
│           0x00000643      4889c6         mov rsi, rax
│           0x00000646      5b             pop rbx
└           0x00000647      ffe1           jmp rcx
```

We need to find the final value of `rax` here.

rcx = 0x312d5334777e7230

rax + 8 = 0x74

rax + 9 = 0x29383f

After `mov qword [rax], rcx`:

rax = 0x29383f74312d5334777e7230

Using python:

```python
>>> import binascii
>>> binascii.unhexlify("29383f74312d5334777e7230").decode("utf-8")[::-1]
'0r~w4S-1t?8)'
```

Fourth part: 0r~w4S-1t?8)


### Flag

We still need to join the parts with underscores and surround them with flag{}:

`flag{tH4T_w45N-T~so_h4rd~huh_0r~w4S-1t?8)}`
