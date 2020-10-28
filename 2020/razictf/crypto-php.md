# Crypto PHP

No GPU Cores Will Be Harmed in the Solving of This Challenge!

Url: http://gold.razictf.ir/PleaseCreateABitcoinAddressForMe/

Tags: web security

## Solution

We are greeted with `I'm not giving you any flags!`. Looking at the source:

```
<!/index.php.bak!>
I'm not giving you any flags!
```

We download the `index.php.bak`:

```php
<?php

function checkAddress($address)
{
    $origbase58 = $address;
    $dec = "0";

    for ($i = 0; $i < strlen($address); $i++)
    {
        $dec = bcadd(bcmul($dec,"58",0),strpos("123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz",substr($address,$i,1)),0);
    }

    $address = "";

    while (bccomp($dec,0) == 1)
    {
        $dv = bcdiv($dec,"16",0);
        $rem = (integer)bcmod($dec,"16");
        $dec = $dv;
        $address = $address.substr("0123456789ABCDEF",$rem,1);
    }

    $address = strrev($address);

    for ($i = 0; $i < strlen($origbase58) && substr($origbase58,$i,1) == "1"; $i++)
    {
        $address = "00".$address;
    }

    if (strlen($address)%2 != 0)
    {
        $address = "0".$address;
    }

    if (strlen($address) != 50)
    {
        return false;
    }

    if (hexdec(substr($address,0,2)) > 0)
    {
        return false;
    }

    return substr(strtoupper(hash("sha256",hash("sha256",pack("H*",substr($address,0,strlen($address)-8)),true))),0,8) == substr($address,strlen($address)-8);
}

$address = "35hK24tcLEWcgNA4JxpvbkNkoAcDGqQPsP";
if (isset ($_GET["PleaseCreateABitcoinAddressForMe"]))
{
$address = $_GET["PleaseCreateABitcoinAddressForMe"];
}

$check = checkAddress($address);
if ($check)
{
	if (substr( $address, 0, 5 ) === "1Razi")
	{
		echo "flag";
	}
}
else
{
	echo "I'm not giving you any flags!";
}

echo $check;

?>
```

We can provide an address through the get parameter `PleaseCreateABitcoinAddressForMe`. It's checked via `checkAdress` which has to return true in order to continue. The provided address has to start with `1Razi`.

Let's analyze what `checkAdress` does. It uses `bcmath` for arbitrary precision calculations. We iterate over the chars of the address and add that to the result of `bcmul($dec, "58", 0)`. After that we have a very large number. Like the first variable name `origbase58` implies, this is base58 -> decimal.

If you want to restore the next step it in python, you need the fractions module, because python converts these numbers to `bignumber`:

```python
>>> n = 42468160281214442798985882180071997964045162989719209072
>>> int(n / 16)
2654260017575902720967102997974858231782517263919218688
>>> (Fraction(n) / 16).numerator
2654260017575902674936617636254499872752822686857450567
>>> int(n / 16) % 16
0
>>> (Fraction(n) / 16 % 16).numerator
7
```

It converts our number from decimal to a hexadecimal string. So you don't actually need it but to initally understand what it is doing it might be helpful.

```python
>>> dec = n
>>> hexchars = "0123456789ABCDEF"
>>> addr = ""
>>> while dec > 0:
...         dv, rem = divmod(dec, 16)
...         dec = Fraction(dv)
...         addr += hexchars[rem.numerator]
...
>>> addr[::-1]
'1BB63666D1882FDD5ED9DA658AFAF56502193C397E54470'
>>> hex(n)[2:].upper()
'1BB63666D1882FDD5ED9DA658AFAF56502193C397E54470'
```

The string is additionally padded with zeros for starting 1s in the address and if the length is odd. The length has to be 50 afterwards and the first two chars (1 byte) has to be zero. Since the required address already starts with a 1 that is converted to zeros this is no problem.

Let's summarize: our adress is converted like this: base58 -> decimal -> hexadecimal string.

### The mistake

The last part is hard to read, storing results in variables we can now see:

```php
<?php
$substr = substr($address,0,strlen($address)-8);
$pack   = pack("H*",$substr);
$hash   = hash("sha256",$pack, true);
$dhash  = hash("sha256",$hash);
$uhash  = strtoupper($dhash);

$substr_one = substr($uhash,0,8);
$substr_two = substr($address,strlen($address)-8);

return $substr_one == $substr_two;
```

While solving this I made a few mistakes. I accidentally swapped `hash` and `dhash` so `substr_one` would be binary. I then searched exhaustively for a 42 char hex string with the sha256 of the sha256 hash starting with only hexadecimal chars (because `substr_two` is the last 8 chars of our hexadecimal address, which can easily control).

Simultaneously I looked for 8 starting chars with the pattern `0+E[0-9+-]\d+` because PHPs loose type checking of the `==` would allow me to set `0e123456 == 000000` or `00e+1234`. This is not the same as comparing the first 8 chars of a hash to zero because a hash doesn't contain `+` and `-`. If you need hashes and inputs like that you can find them here: https://github.com/spaze/hashes

Yes, the second search pattern doesn't differ from the hexadecimal search, but it was actually my first guess and I added the hexadecimal search later.

I constantly thought: This can't be right! The challenge said `No GPU Cores Will Be Harmed in the Solving of This Challenge!`.

I actually managed to find a possible input after roughly 30 minutes of computing. I then wrote a function to reverse the adresse from hexadecimal back to base58. Which was useless because I swapped the hashing order.

Then I realized my second mistake: I didn't see that the address has to start with `1Razi`.

### The enlightment

So I started from scratch. I reanalyzed my approach.

I realized I can pass nearly any address into the function and just have to set the last 8 chars of the hexadecimal address to the first 8 chars of the hashing result and reverse that.

I added this before the return and a `var_dump` before the first if-return to correct the length of the address:

```php
<?php
$first = substr(strtoupper(hash("sha256",hash("sha256",pack("H*",substr($address,0,strlen($address)-8)),true))),0,8);
echo "full=" . $address . "\n";
echo "need=" . $first . "\n";
```

We can paste the generated lines in this solve script:

```python
from baseconv import base58

full='0004A65C610EF028FAA769414559699B832B804515821C70FA'
need='744AB939'

print("1" + base58.encode(int(full[:-8] + need, 16)))
```

And we get: `1RazitcLEWcgNA4JxpvbkNkoAcDGUv7aG`.

If we request the page with that as a get parameter we get the flag.

After all it was really easy. Still took me hours to realize my mistakes.
