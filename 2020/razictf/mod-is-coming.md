# Mod Is Coming

## Task

It's actually pretty simple when you think about it.

Tags: steganography

File: Mod Is Coming.zip

## Solution

Inside is a `enc.png` with horizontal colored lines. There is also a `script.py` that created it.

Inside the script we have a `encrypt` function and three helper functions. The task is now to open the encrypted image and reverse the process.

```python
def encrypt(s):
    c_p = []
    for i in range(10,21):
        for j in range(i+1,21):
            if f2(i,j) == 1:
                c_p.append([i,j])

    rand = random.randint(0,len(c_p))
    k = f1(c_p[rand], [len(s), len(s)*3])
    while k == 0:
        rand = random.randint(0,len(c_p))
        k = f1(c_p[rand], [len(s), len(s)*3])

    img = Image.new('RGB', (len(s)*3, len(s)), color = 'white')
    image = array(img)
    x, y, z = image.shape

    for a in range (0, x):
        for b in range (0, y):
            p = image[a, b]
            p[0] = ((k-10) * ord(s[a])) % 251
            p[1] = (k * ord(s[a])) % 251
            p[2] = ((k+10) * ord(s[a])) % 251
            image[a][b] = p

    enc = Image.fromarray(image)
    enc.save('enc.png')

f = open("secretmsg.txt", "r")
encrypt(f.read())
```

Step 1: a lot of tuples that match the condition `f2(i, j) == 1` are appened to an array.
Step 2: a random entry is chosen to generate a key `f1(c_p[rand], [len(s), len(s)*3])`
Step 3: an image is created with the height of the length of the secret.
Step 4: iterating over the rows in the image, each row is set to an RGB value calculated from the key and the `ord()` of a secret char.

First we need the length of the secret, which is the height of the image:

```bash
$ exiftool -S -ImageHeight enc.png
ImageHeight: 331
```

Solving step 1 and 2. We generate a list of all possible keys:

```python
c_p = []
for i in range(10,21):
  for j in range(i+1,21):
    if f2(i,j) == 1:
      c_p.append(f1([i,j], [len(s), len(s)*3]))

c_p = set(c_p)
```

Solving step 3. We just open the encrypted image as RGB:

```python
im = Image.open("enc.png").convert("RGB")
ima = array(im)
y, x = im.size
```

Solving step 4. We need to find the correct key.

The easiest way to do this is to iterate over all possible keys and all ascii chars, compute the value just like the script did and compare the pixel values with the results:

```python
ks = []
  for k in c_p:
    for a in range(0, x):
      for c in range(256):
        p = [0,0,0]
        p[0] = ((k-10) * c) % 251
        p[1] = ( k     * c) % 251
        p[2] = ((k+10) * c) % 251
        v = ima[a,0]
        if p == [v[0], v[1], v[2]]:
          ks.append(k)

# ks now contains all possible keys that decrypt at least one character

for k in set(ks):
  for a in range(0, x):
    v = ima[a,0]
    for n in range(256):
      res = (v[1] + (251 * n)) / k
      # result is only valid if it's not a float
      if res == int(res) and res < 256:
        print(chr(int(res)), end="")
print()
```

Running it:

`Chaos isn't a pit. Chaos is a ladder, Many who try to climb it fail, and never get to try again, the fall breaks them. And some are given a chance to climb, but they refuse. They cling to the realm, or the gods, or love ... illusions. Only the ladder is real, the climb is all there is. RaziCTF{7h3_script_1s_d4rk_4nd_full_0f_m0ds}`
