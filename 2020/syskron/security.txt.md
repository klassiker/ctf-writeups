# Security.txt

## Task

The security.txt draft got updated (https://tools.ietf.org/html/draft-foudil-securitytxt-10).

Is Senork's file still up-to-date? https://www.senork.de/.well-known/security.txt

Tags: best-practices

## Solution

Looking at the security.txt file we notice the email address `psirt@bb-industry.cz` which seems to non-compliant with the spec:

`As per section 4 of [RFC2142], there is an existing convention of using the <SECURITY@domain> email address for communications regarding security vulnerabilities.`

There is nothing else obvious in there so we try to find information about this address.

Since the file is signed by a pgp key and communications regarding security vulnerabilities should be encrypted we try to find information about the key.

So we search for this mail address with a pgp keyserver and find:

`BB Industry a.s. PSIRT (syskronCTF{Wh0-put3-flag3-1nto-0penPGP-key3???}) <psirt@bb-industry.cz>`
