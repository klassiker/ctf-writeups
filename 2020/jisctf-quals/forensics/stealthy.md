# Stealthy

## Task

Find out whats going on our network?

File: capture.pcapng

## Solution

After filtering almost everything, looking at all the fields (TLS SNI for example), this was my filter:

`not (udp or http or http2 or ftp or ftp-data or tls or tcp.flags in {0x2 0x10 0x11 0x12 0x18})`

What you are left with are some simple ICMP requests and replies.

We can filter `icmp` now. Let's separate `request` and `reply` now:

`icmp.type == 8`

You can see there is a `J` in the first packet, `I` in the second, `S` in the third.

The field is `ip.ttl`.

We can now extract them with magic (tshark + xxd reverse):

```bash
$ printf '%x' $(tshark -r capture.pcapng -Tfields -e 'ip.ttl' 'icmp.type == 8') | xxd -p -r;echo
JISCTF{M4LW4R3_3XF1LT3R4T10N_US1NG_1CMP_TTL}
```
