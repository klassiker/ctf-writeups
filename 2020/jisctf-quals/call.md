# Call

## Task

Decrypt me :

00110000910000FF2E547419646687CFA0F41CA4032993D321D5B8414D9BD348D1397C1293CE63C458753AB3915028B44901

## Solution

You will notice that you can separate the string in to parts:

- 00110000910000
- FF2E547419646687CFA0F41CA4032993D321D5B8414D9BD348D1397C1293CE63C458753AB3915028B44901

Searching for the first one results in `PDU encode/decode` and `How to send long sms`.

So we look for a SMS PDU Decoder.

Like this one: https://www.diafaan.com/sms-tutorials/gsm-modem-tutorial/online-sms-pdu-decoder/

```
To:	+
Message: The flag is : JISCTF{SMS_ENCODING_FUNNY_!!!}
```
