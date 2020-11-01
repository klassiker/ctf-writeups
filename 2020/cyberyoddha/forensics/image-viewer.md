# Image Viewer

## Task

My friend took this image in a cool place

File: shoob_2.jpeg

## Solution

The flag is in the exif:

```bash
$ exiftool -S -LensSerialNumber shoob_2.jpeg
LensSerialNumber: CYCTF{h3h3h3_1m@g3_M3t@d@t@_v13w3r_ICU}
```
