# chasing a lock

## Task

as locks are so popular many will chase them but why? maybe a flag :)

File: chasing.zip

Tags: android security

## Solution

Inside the zip is an apk. We use apktool to for now:

```bash
apktool d --no-src
```

Looking through `res`, nothing useful there.

We continue with decompiling the `classes.dex`:

```bash
$ jadx -ds src --no-res --show-bad-code --no-replace-consts --escape-unicode --deobf classes.dex
INFO  - loading ...
INFO  - processing ...
ERROR - finished with errors, count: 3
```

We now have a `src/com/example/razictf_2/` folder. We have a `MainActivity`, a `switcher` and 5 classes.

I reconstructed the project to understand what's happening. We create our own `switcher` class with a main function and copy the code from the decompiled file.

```java
public static String run(int paramInt) {
    if (paramInt == 0) {
        String str;
        str = " " + (new a1()).run();
        str = str + (new a2()).run();
        str = str + (new a3()).run();
        str = str + (new a4()).run();
        str = str + (new a5()).run();
        return str;
    }
    return null;
}
```

I already refactored it to make it more readable. There were some `StringBuilder` instances and `paramInt` was passed on to the run functions but never used.

This function probably builds the flag. It is called from the `MainActivity`:

```java
imageButton.setOnClickListener(new View.OnClickListener() {
    public void onClick(View view) {
        TextView textView = (TextView) MainActivity.this.findViewById(2131165190);
        int parseInt = Integer.parseInt(textView.getText().toString());
        if (parseInt == 0 || parseInt < 0) {
            textView.setText("0");
            return;
        }
        int i = parseInt - 1;
        String run = new switcher().run(i);
        if (run != null) {
            ((TextView) MainActivity.this.findViewById(2131165187)).setText(run);
        }
        textView.setText(String.valueOf(i));
    }
});
```

The content of a `TextView` is converted to an integer, if it's `0` the new text ist `0`, but `switcher.run` only builds the string if `paramInt` is `0`. Let's just ignore that file and continue reconstructing.

We need all 5 helper classes now.

`a1` to `a4` are pretty simple to understand.

`a5` contains multiple errors: `/* JADX WARNING: type inference failed for: r5v35 */`. There is also a lot of source code with nested loops, arrays and chars moving around. Inside the innermost loop a string is built an hashed with md5. If the hash is equal to `b469f80f05290ed415770ea56e69a476` it is returned.

The type of `r5` couldn't be infered so we have to fix that. Comment it out. There is a new problem now in the innermost try-catch block:

```java
try {
   // r5 = str5;
    sb2.append(str5);
    sb2.append(hexString);
    hexString = sb2.toString();
    str3 = str5;
    strArr18 = strArr;
} catch (Exception e) {
    str = r5; // r5 has not been declared
    e.printStackTrace();
    i12++;
    str3 = str;
    strArr8 = strArr4;
    strArr5 = strArr3;
    strArr6 = strArr2;
    strArr14 = strArr;
    i3 = 3;
}
```

Since `r5` was set to `str5` we just set `str = str5`.

Inside almost all of the other catch blocks the variable `e` can't be used but we can replace that with the variable containing the exception (`e2` to `e6`).

Since we looked at the whole code we can safely execute it with our previously created main function. After a lot of bruteforcing from `a5` we get: `RaziCTF{IN_HATE_0F_RUNN!NG_L0CK5}`
