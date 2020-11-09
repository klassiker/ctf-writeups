# Password Pandemonium

## Task

You're looking to book a flight to Florida with the totally-legit new budget airline, Oceanic Airlines! All you need to do is create an account! Should be pretty easy, right?

...right?

https://pandemonium.web.2020.sunshinectf.org

## Solution

We have to register an account. Easier said than done, we have funny password restrictions.

 - Must be long enough
 - Must include letters
 - Equal amount of lower and uppercase
 - At least 3 special chars
 - Must include an emoji
 - Amount of numbers has to be prime
 - Has to be valid Javascript that evaluates to true
 - Has to be a palindrome
 - MD5 hash has to start with a number

I'm not explaining you every step here. The following string matches the criteria:

`2||"Rs"&&"😂"&&"sR"||2`

We use `&&` and `||` as special chars, an emoji in the middle, two numbers and our chars matching the palindrome. Change the numbers or letters until you get the required MD5.

`sun{Pal1ndr0m1c_EcMaScRiPt}`
