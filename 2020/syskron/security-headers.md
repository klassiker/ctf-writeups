# Security headers

## Task

Can you please check the security-relevant HTTP response headers on www.senork.de. Do they reflect current best practices?

Tags: web

## Solution

We want to see the headers (-I) and follow potential redirects (-L):

```bash
curl -IL http://www.senork.de/
```

On the second redirect we find the header `flag-policy` which contains the flag.
