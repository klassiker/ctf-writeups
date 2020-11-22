# Malicious

## Task

File: attachment.docx

## Solution

```bash
$ file attachment.docx
attachment.docx: Microsoft OOXML
```

We use `binwalk` to extract the contents:

```bash
$ binwalk -e attachment.docx
```

We find an `image.png`. Don't get fooled here:

```bash
$ file image.png
image.png: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, Exif Standard: [TIFF image data, big-endian, direntries=7, manufacturer=BeFunky, orientation=upper-left, xresolution=106, yresolution=114, resolutionunit=2, software=BeFunky Photo Editor], baseline, precision 8, 642x76, components 3
```

But that doesn't matter. The image shows a barcode so we use this tool to read it: https://www.onlinebarcodereader.com/

Content: `JISCTF{B4RC0D3_1M4G3_2019}`
