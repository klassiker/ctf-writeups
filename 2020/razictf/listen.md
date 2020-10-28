# Listen

## Task

Listen carefully, what do you hear? Look closely, what do you see? To Submit the flag, put it in UPPERCASE and in this format RaziCTF{}. like this: RaziCTF{FLAG}

File: enc.mp3

Tags: steganography

## Solution

After listening to the morse code we open `audacity` and click on `enc` (you can find it on the left side of the audio track right next to the cross and on top the controls) and select spectrogram.

We now can see the beeps and bops. Write it down as `-` and `.` and throw it in your favorite morse decoder.

`THEREALFLAGISS1MPL3M0RS3  THI3ISN0T4R34LFL4GFOCUSUE`

The flag is `RaziCTF{S1MPL3M0RS3}`

Using sox would also be possible:

```bash
$ sox enc.mp3 -n spectrogram
$ view spectrogram
```
