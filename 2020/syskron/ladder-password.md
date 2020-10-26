# Ladder password

## Task

A BB engineer encoded her password in ladder logic. Each "r" can be either 0 or 1 and represents a character in her password. "r1" is the first character, "r2" the second one, and so on. We assume that n1 is a small value and n3 is big. n2 may be between n1 and n3.

Please decode the password to demonstrate that this technique of storing passwords is insecure.

Flag format: r1r2r3…r9r10 (e.g., 1101…001).

Be aware: BB deployed brute-force detection. You only have 3 attempts until lockout!

File: ladder-password.png

Tags: plc-programming

## Solution

If you don't know how to read ladder logic you can use a simulator/editor. There is an online version at: https://www.plcfiddle.com/

Reconstructing the image and adjusting n1,n2,n3 according to the task we get:

0011100011
