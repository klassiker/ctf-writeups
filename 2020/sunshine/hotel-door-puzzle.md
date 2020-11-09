# Hotel Door Puzzle

## Task

I thought I'd come down to Orlando on a vacation. I thought I left work behind me! What's at my hotel door when I show up? A Silly reverse engineering puzzle to get into my hotel room! I thought I was supposed to relax this weekend. Instead of me doing it, I hired you to solve it for me. Let me into my hotel room and you'll get some free internet points!

File: hotel_key_puzzle

Tags: reverse engineering

## Solution

We are presented with a `main` that calls a `check_flag` function with user input. The length has to be `0x1d` and then we have a lot of nested ifs that check char by char and modify some chars by an offset.

I found the easiest way to do this is loading it up in `ghidra`, decompiling it and to manually keep track of each char.

This is what my notes looked like:

```
0x00 "s":-3:-8:"h":+3:"k"
0x01 -2:"s"
0x02 "n":"n":"n":"n":"n"
0x03 "{":+5:-9:-9:"n"
0x04 "b":"b":+5:"g"
0x05 +5:"8"
0x06 +3:"o":-9:"f":+5:"k":-7:"d"
0x07 "l":-9:"c":+8:"k"
0x08 +3:+4:-9:"f"
0x09 +1:+2:-7:","
0x0a "p":+7:"w"
0x0b +7:"<"
0x0c -1:",":-2:"*"
0x0d "r":-4:-10:"d"
0x0e "u":"u":"u":"u"
0x0f "n":"n":"n":-3:+3
0x10 "n":+7:"u":-4:"q":-4
0x11 +6:-8:+9:-5:-6:"-"
0x12 "n":-9:"e"
0x13 "6":-2:-10:"*"
0x14 -8:"%":-4:+4:"%":+4:")"
0x15 "q":"q"
0x16 +5:"z":-6:"t"
0x17 "1":-5:","
0x18 -10:-8:+8:"Y"
0x19 +5:+7:"w"
0x1a -6:+7:"m"
0x1b "y"
0x1c "}":+4:-8:"y"
```

There are some additional notes on variables that were checked multiple times, I stopped writing them up at some point. I kept track of already known chars to, maybe they would be added together. I should have looked for that earlier since this is not the case. Could have been less writing.

Just remove unwanted data and reverse the calculation:

```
0x00 s "s"
0x01 u -2:"s"
0x02 n "n"
0x03 { "{"
0x04 b "b"
0x05 3 +5:"8"
0x06 l +3:"o"
0x07 l "l"
0x08 h +3:+4:-9:"f"
0x09 0 +1:+2:-7:","
0x0a p "p"
0x0b 5 +7:"<"
0x0c - -1:","
0x0d r "r"
0x0e u "u"
0x0f n "n"
0x10 n "n"
0x11 1 +6:-8:+9:-5:-6:"-"
0x12 n "n"
0x13 6 "6"
0x14 - -8:"%"
0x15 q "q"
0x16 u +5:"z"
0x17 1 "1"
0x18 c -10:-8:+8:"Y"
0x19 k +5:+7:"w"
0x1a l -6:+7:"m"
0x1b y "y"
0x1c } "}"
```

And we get the flag: `sun{b3llh0p5-runn1n6-qu1ckly}`.

What a waste of time.
