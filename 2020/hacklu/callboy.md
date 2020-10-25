# Callboy

## Task

Categories: misc

Difficulty: baby

Have you ever called a Callboy? No!? Then you should definitely try it. To make it a pleasant experience for you, we have recorded a call with our Callboy to help you get started, so that there is no embarrassing silence between you.

PS: do not forget the to wrap flag{} around the secret

File: Callboy.zip

## Solution

We unpack the zip and get a Callboy.pcapng. We open that with wireshark.

The challenge mentions a call, so we click on `Telephony -> VoIP Calls`. We select the stream and click `Play stream`. Select the first first audio and the correct sound device. Click play and listen closely.

flag{call_me_baby_1337_more_times}
