# Calculator

## Task

Kevin has created a cool calculator that can perform almost any mathematical operation! It seems that he might have done this the lazy way though... He's also hidden a flag variable somewhere in the code.

https://calculator.challenges.nactf.com/

## Solution

The lazy way probably means using `eval` on user input, so let's provide `$flag`: `$flag = nactf{ev1l_eval}`
