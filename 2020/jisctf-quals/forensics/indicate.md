# Indicate

## Task

Our SOC team got that our network was compromised but the don't know how to indicate what is going one. They got the following strings from the SIEM solution:

001351e2ab912ce17ce9ab2c7465f1a265aa975c879e863dea06e177b62d8750
fe1a602aadba2e52b3b2cf82139c4d10c9fc14889435e0bd9aa54d30760fd1db
352656186eb3bb7e7495fa0922a4defce93bc2213c4d37adc6e42b59debee09e
d672df1dea88e3ad2159b9c7b2df1dbd39b912e648c5097800f48fbf95cadd70
352656186eb3bb7e7495fa0922a4defce93bc2213c4d37adc6e42b59debee09e
0e6957de845ce5a1b0a73e91d12383714bda0fac66b13002170c1ff73426b82a
069cf46549f856b7fc266feab68968ad1d5ec4569f6326d71e999526b721ee0c
39759c4fd7193a29cda8ea8714e690c8b5ec374b659a3a1a3bc402c1ba20364a
c25b0ab2413a7b300fc06d5dc5ec9807ec21372b27a5162fa2eb9729b84dd28c
213d537d2f63c70249a4244c394d9c364476ca6fd1ee04d5dda7ddaaf60a04e7
e366a166b172f225d842c0662f5cb261c0d7b50430dbd392f3eb33249fdb375c


Can you help them to solve the issue?

## Solution

First thing to do is looking these (probably) hashes up with `virustotal`.

After I found nothing I used https://www.hybrid-analysis.com/

It told me the first hash was in an `Unknown Files Collection`. All the other hashes are in there as well, except one.

Looking that up gives `JISCTF2020.pdf`.

In the `Screenshots` section we click on `Show more` and find a screenshot of the second page with the flag:

`JISCTF{IOC-Search-Using-OSINT}`
