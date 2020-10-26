# DoS attack

## Task

One customer of Senork Vertriebs GmbH reports that some older Siemens devices repeatedly crash. We looked into it and it seems that there is some malicious network traffic that triggers a DoS condition. Can you please identify the malware used in the DoS attack? We attached the relevant network traffic.

Flag format: syskronCTF{name-of-the-malware}

File: dos-attack.pcap

Tags: packet-analysis

## Solution

There is not much interesting in the traffic, just one packet that repeats over and over again.

It's a DNS Query without questions. Relevant information might be:

 - Transaction ID: 0x1149
 - Destination Port: 50000

Searching for `siemens denial of service malware` we find an article about a new malware similar to the `Industroyer` which also targets UDP-Port 50000.
