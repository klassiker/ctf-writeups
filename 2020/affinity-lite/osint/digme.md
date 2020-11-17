# DIGme

## Task

www.affinityctf.com

## Solution

`dig` is a command line utility for DNS lookups. You can also use `host`. A good entry to look for is `TXT`:

```bash
$ host -t TXT www.affinityctf.com
www.affinityctf.com descriptive text "QUZGQ1RGe0hlcmUnNXkwdXJUcmVhNXVyZX0="
$ echo "QUZGQ1RGe0hlcmUnNXkwdXJUcmVhNXVyZX0=" | base64 -d
AFFCTF{Here'5y0urTrea5ure}
```
