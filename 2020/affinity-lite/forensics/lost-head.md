# Lost Head

## Task

We lost some data when the connection closed, can you recover something?

File: lostHead.pcapng

## Solution

We can open this file with wireshark and search for interesting information.

First thing I do is separating `udp` and `tcp`. After filtering each `udp` protocol until nothing is left I start with `tcp`.

First I look for `http`, which was successfull this time.

We could also use `tshark` to extract information, but that's not a good idea when you don't know where the data is hidden.

Solution in this case:

```bash
$ tshark -r lostHead.pcapng -Tfields -e 'http.response.line' 'http.response.code == 200 and not ocsp'
Server: nginx/1.14.1\r\n,Date: Mon, 02 Nov 2020 12:08:41 GMT\r\n,Content-Type: text/html; charset=UTF-8\r\n,Transfer-Encoding: chunked\r\n,Connection: keep-alive\r\n,X-Powered-By: PHP/7.2.24\r\n,X-Affinity: AFFCTF{DonT_TRusT_h34d3R2}\r\n
```
