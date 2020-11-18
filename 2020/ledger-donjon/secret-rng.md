# Secret RNG

## Task

Our previous implementation was flawed!

I don't want to use the crypto/rand generator, as it is probably not quantum resistant! Instead, I modified all the rngCooked values in the previous generator.

There are so many values! It is impossible to retrieve them.

ots-sig.donjon-ctf.io:4001

File: challenge.go

## Solution

Connecting to the address:

```bash
$ nc ots-sig.donjon-ctf.io 4001
Public key: [REDACTED]
Enter signature:
You failed! Private key was: [REDACTED]

Public key: [REDACTED]
Enter signature: ^C
```

This challenge is the continuation of `One Time-Based Signature`. This time we have this source:

```go
package main

import (
	"bufio"
	"crypto/sha256"
	"encoding/base64"
	"fmt"
	"io/ioutil"
	"os"
	"strings"
	"time"

	"./math/rand" // same as default implementation, with different rngCooked array
	"github.com/dchest/wots"
)

const defaultFlag string = "CTF{xxx}"

func main() {
	var message = []byte("Sign me if you can")

	// Not so secure seed, but prng internals are secret
	rng := rand.New(rand.NewSource(time.Now().UnixNano()))

	for {
		var ots = wots.NewScheme(sha256.New, rng)
		priv, pub, _ := ots.GenerateKeyPair()

		fmt.Println("Public key:", base64.StdEncoding.EncodeToString(pub))

		reader := bufio.NewReader(os.Stdin)
		fmt.Print("Enter signature: ")
		text, err := reader.ReadString('\n')
		if err != nil {
			fmt.Println("Error occurred. Please try again later.")
			return
		}

		text = strings.TrimSuffix(text, "\n")
		signature, err := base64.StdEncoding.DecodeString(text)
		if err != nil {
			return
		}

		if ots.Verify(pub, message, signature) {
			fmt.Print("Congratulations! Flag: ")

			flag, err := ioutil.ReadFile("secret")
			if err != nil {
				fmt.Println(defaultFlag)
			} else {
				fmt.Println(string(flag))
			}
		} else {
			fmt.Println("You failed! Private key was:", base64.StdEncoding.EncodeToString(priv))
			fmt.Println()
		}
	}
}
```

The time is still used to seed the PRNG but this time with all bits from `time.Now().UnixNano()`. It's also noted that we use this: `"./math/rand" // same as default implementation, with different rngCooked array`.

In `One Time-Based Signature` we only had one time-based public key, now we have an unlimited amount of private keys that are generated from a PRNG that's seeded only once.

If you've never attacked a PRNG before like me you might want to read this: https://insomniasec.com/cdn-assets/Not_So_Random_-_Exploiting_Unsafe_Random_Number_Generator_Use.pdf

The concept is simple:
 - in a PRNG we have seeds, a state and a period
 - the seed is used to create the initial state
 - the current state contains the internal properties of the PRNG
 - the period describes the length of all possible outputs before it's repeated

We are going to look at the `math/rand` source and create our own local version of the package to manipulate it and find out which functions are actually called, where to attack and what this has to do with `rngCooked`.

We need: `math/rand/rng.go` and `math/rand/rand.go`.

If you try this challenge yourself you might copy all from the `math/rand` folder, but these are the ones that are actually used in the challenge setup. They are typically in `/usr/lib/go/src/` or here: https://golang.org/src/math/rand/

This is the relevant part merged from both files:

```go
// NewSource returns a new pseudo-random Source seeded with the given value.
// Unlike the default Source used by top-level functions, this source is not
// safe for concurrent use by multiple goroutines.
func NewSource(seed int64) Source {
	var rng rngSource
	rng.Seed(seed)
	return &rng
}

type rngSource struct {
	tap  int           // index into vec
	feed int           // index into vec
	vec  [rngLen]int64 // current feedback register
}

const (
	rngLen   = 607
	rngTap   = 273
	rngMax   = 1 << 63
	rngMask  = rngMax - 1
	int32max = (1 << 31) - 1
)

// Seed uses the provided seed value to initialize the generator to a deterministic state.
func (rng *rngSource) Seed(seed int64) {
	rng.tap = 0
	rng.feed = rngLen - rngTap

	seed = seed % int32max
	if seed < 0 {
		seed += int32max
	}
	if seed == 0 {
		seed = 89482311
	}

	x := int32(seed)
	for i := -20; i < rngLen; i++ {
		x = seedrand(x)
		if i >= 0 {
			var u int64
			u = int64(x) << 40
			x = seedrand(x)
			u ^= int64(x) << 20
			x = seedrand(x)
			u ^= int64(x)
			u ^= rngCooked[i]
			rng.vec[i] = u
		}
	}
}

// seed rng x[n+1] = 48271 * x[n] mod (2**31 - 1)
func seedrand(x int32) int32 {
	const (
		A = 48271
		Q = 44488
		R = 3399
	)

	hi := x / Q
	lo := x % Q
	x = A*lo - R*hi
	if x < 0 {
		x += int32max
	}
	return x
}

// Uint64 returns a non-negative pseudo-random 64-bit integer as an uint64.
func (rng *rngSource) Uint64() uint64 {
	rng.tap--
	if rng.tap < 0 {
		rng.tap += rngLen
	}

	rng.feed--
	if rng.feed < 0 {
		rng.feed += rngLen
	}

	x := rng.vec[rng.feed] + rng.vec[rng.tap]
	rng.vec[rng.feed] = x
	return uint64(x)
}
```

The function `seedrand` can be ignored since it's not modified in the challenge.

Things we notice:
  - the `int64 seed` is converted `seed % int32max`, so we don't have to guess 64 bits, only 32 bits from which some can be guessed based on the time (this is the seed)
  - special seeds like `seed < 0` (which will not happen with time) or `seed == 0` (which could happen since the modulo can be zero, but can't take advantage of this, chances are 1 : int32max) are used
  - after a lot of deterministic calculation based on the seed we `XOR` the resulting value with `rngCooked`, which we don't have (this is the initial state)
  - the resulting value is stored in `rng.vec` and read with `rng.tap` and `rng.feed` properties (this is the current state)
  - since `rng.tap` and `rng.feed` are rolled over to `rngLen`
  - values from these positions are added and stored back at `rng.feed` in the `rng.vec` (this increases the period)

Next we need to look at how the `ots.GenerateKeyPair()` call uses the provide PRNG to generate the private key.

Here are the relevant parts of `wots.go` from: https://github.com/dchest/wots

```go
// Scheme represents one-time signature signing/verification configuration.
type Scheme struct {
	blockSize int
	hashFunc  func() hash.Hash
	rand      io.Reader
}

// NewScheme returns a new signing/verification scheme from the given function
// returning hash.Hash type and a random byte reader (must be cryptographically
// secure, such as crypto/rand.Reader).
//
// The hash function output size must have minimum 16 and maximum 128 bytes,
// otherwise GenerateKeyPair method will always return error.
func NewScheme(h func() hash.Hash, rand io.Reader) *Scheme {
	return &Scheme{
		blockSize: h().Size(),
		hashFunc:  h,
		rand:      rand,
	}
}

// PrivateKeySize returns private key size in bytes.
func (s *Scheme) PrivateKeySize() int { return (s.blockSize + 2) * s.blockSize }

// PublicKeySize returns public key size in bytes.
func (s *Scheme) PublicKeySize() int { return s.blockSize }

// SignatureSize returns signature size in bytes.
func (s *Scheme) SignatureSize() int { return (s.blockSize+2)*s.blockSize + s.blockSize }

// PublicKey represents a public key.
type PublicKey []byte

// PrivateKey represents a private key.
type PrivateKey []byte

// hashBlock returns in hashed the given number of times: H(...H(in)).
// If times is 0, returns a copy of input without hashing it.
func hashBlock(h hash.Hash, in []byte, times int) (out []byte) {
	out = append(out, in...)
	for i := 0; i < times; i++ {
		h.Reset()
		h.Write(out)
		out = h.Sum(out[:0])
	}
	return
}

// GenerateKeyPair generates a new private and public key pair.
func (s *Scheme) GenerateKeyPair() (PrivateKey, PublicKey, error) {
	if s.blockSize < 16 || s.blockSize > 128 {
		return nil, nil, errors.New("wots: wrong hash output size")
	}
	// Generate random private key.
	privateKey := make([]byte, s.PrivateKeySize())
	if _, err := io.ReadFull(s.rand, privateKey); err != nil {
		return nil, nil, err
	}
	publicKey, err := s.PublicKeyFromPrivate(privateKey)
	if err != nil {
		return nil, nil, err
	}
	return privateKey, publicKey, nil
}

// PublicKeyFromPrivate returns a public key corresponding to the given private key.
func (s *Scheme) PublicKeyFromPrivate(privateKey PrivateKey) (PublicKey, error) {
	if len(privateKey) != s.PrivateKeySize() {
		return nil, errors.New("wots: private key size doesn't match the scheme")
	}

	// Create public key from private key.
	keyHash := s.hashFunc()
	blockHash := s.hashFunc()
	for i := 0; i < len(privateKey); i += s.blockSize {
		keyHash.Write(hashBlock(blockHash, privateKey[i:i+s.blockSize], 256))
	}
	return keyHash.Sum(nil), nil
}
```

The concept of `wots` is the `Winternitz One-Time Signature` described here: https://cryptoservices.github.io/quantum/2015/12/04/one-time-signatures.html

The security is based on the security of the one-way function, in this case sha256. Each block is hashed 256 times, the resulting hashes are also hashed. We can't attack that.

Things we notice instead:
 - we use `io.ReadFull` to read the random values into `privateKey`, which is a byte array of the private key size (in this case 1088)
 - that private key is returned, so the base64 encoded private key we receive from the server are the raw results from the read call

Now we need to look at the `io.Reader` in `math/rand/rand.go`:

```go
// New returns a new Rand that uses random values from src
// to generate other random values.
func New(src Source) *Rand {
	s64, _ := src.(Source64)
	return &Rand{src: src, s64: s64}
}

// A Rand is a source of random numbers.
type Rand struct {
	src Source
	s64 Source64 // non-nil if src is source64

	// readVal contains remainder of 63-bit integer used for bytes
	// generation during most recent Read call.
	// It is saved so next Read call can start where the previous
	// one finished.
	readVal int64
	// readPos indicates the number of low-order bytes of readVal
	// that are still valid.
	readPos int8
}

func (r *Rand) Read(p []byte) (n int, err error) {
	if lk, ok := r.src.(*lockedSource); ok {
		return lk.read(p, &r.readVal, &r.readPos)
	}
	return read(p, r.src, &r.readVal, &r.readPos)
}

func read(p []byte, src Source, readVal *int64, readPos *int8) (n int, err error) {
	pos := *readPos
	val := *readVal
	rng, _ := src.(*rngSource)
	for n = 0; n < len(p); n++ {
		if pos == 0 {
			if rng != nil {
				val = rng.Int63()
			} else {
				val = src.Int63()
			}
			pos = 7
		}
		p[n] = byte(val)
		val >>= 8
		pos--
	}
	*readPos = pos
	*readVal = val
	return
}

// Int63 returns a non-negative pseudo-random 63-bit integer as an int64.
func (rng *rngSource) Int63() int64 {
	return int64(rng.Uint64() & rngMask)
}
```

We aren't using a locked source, so we can ingore that.

Things we notice:
 - the `rng.Uint64()` value is ANDed with the `rngMask` to get an `int64`
 - that value is read one byte at a time and stored in the output byte array
 - we only use 7 of 8 bytes of the generated integer
 - if we don't read a multiple of 7 the remaining part of `val` is stored in `readVal` and `readPos` of `rand.Rand`

The `func (r *Rand) Read(p []byte)` only uses 7 of the 8 bytes. Our `PrivateKeySize` is `1088` bytes, we therefore have the last 3 bytes from another call of `func (rng *RngSource) Uint64()` since `1088 % 7 = 3`. If we generate 7 keys our `readPos` is back at 0, the next call will read a fresh value since `x * 7 % 7 = 0`. This makes things easier I guess.

Now we know everything we need to start solving this challenge.

## Actually solving it

We need to attack the internals of golangs `math/rand` PRNG.

The server leaks us the generated bytes with the privatekey, we can use them to restore the internal state of the generator.

First thing we do: Removing all unnecessary functions from the copied `math/rand` package after removing files we don't need

Next:
 - export all fields, functions and variables from our `math/rand` version to access them in our main function
 - write a function to get the `uint64` back from a 7 byte array
 - initialize our own PRNG to store the obtained state in it

This is my reverse of the `uint64` problem:

```go
func restoreUint64(a []byte) uint64 {
        var val int64
        for i := len(a) - 1; i >= 0; i-- {
                val |= int64(a[i]) & rngMask

                if i > 0 {
                        val <<= 8
                }
        }
        return uint64(val)
}
```

Because working with the remote data makes debugging harder I started off with a time-based PRNG and my 'attack' PRNG seeded with zero.

I then obtained `rngLen` bytes from it to write the calculated `uint64` into the `vec` field of my 'attack' PRNG.

This would look something like this:

```go
func main() {
  timeRand := rand.New(rand.NewSource(time.Now().UnixNano()))

  var leakedBytes []byte
  for i := 0; i < rngLen; i++ {
    random := make([]byte, 7)
    timeRand.Read(random)
    leakedBytes = append(leakedBytes, random...)
  }

  myRand := rand.New(rand.NewSource(0))
  mySource := myRand.S64.(*rand.RngSource)

  for i := 0; i < len(leakedBytes); i += 7 {
    leak := leakedBytes[i : i+7]

    mySource.Uint64() // calling here to forward feed and tap
    mySource.Vec[mySource.Feed] = int64(restoreUint64(leak))
  }

  for i := 0; i < rngLen; i++ {
    random, mybyte := make([]byte, 7), make([]byte, 7)
    timeRand.Read(random)
    mySource.Read(mybyte)
    if !equals(random, mybyte) {
      log.Fatal("Not matching!", random, mybyte)
    }
  }
}
```

After encountering some bugs I thought that maybe an `int64` overflow leading to negative values in the original `vec` was the problem so I simply used a modified array `[rngLen][]int64` so that I could store more possible values for a single position in there. After `rngLen` rounds of restoring I tried adding the stored values from my own `vec` to compare them to the expected ones. Since there were now multiple values I calculated each possible combination using the cartesian product of `vec[feed]` and `vec[tap]`.

I then fixed my bug and removed all of that again since it's not needed.

After verifying that everything works I changed the first loop to append a generated private key to the `leakedBytes`.

That worked too. So I replaced that with a loop that read 7 private keys from the remote host and added code that would then generate the eight private key and compare the pubkey to the one received from the remote.

If they are equal I sign the message.

The code now looks like this:

```go
package main

import (
	"./math/rand"
	"bufio"
	"crypto/sha256"
	"encoding/base64"
	"github.com/dchest/wots"
	"log"
	"net"
	"strings"
)

const (
	rngMax   = 1 << 63
	rngMask  = rngMax - 1
)

func check(err error) {
	if err != nil {
		log.Fatal(err)
	}
}

func removeNewline(r string) string {
	return strings.TrimSuffix(r, "\n")
}

func getPubKey(r string) string {
	return removeNewline(strings.TrimPrefix(r, "Public key: "))
}

func getPrivKey(r string) string {
	return removeNewline(strings.TrimPrefix(r, "You failed! Private key was: "))
}

func restoreUint64(a []byte) uint64 {
	var val int64
	for i := len(a) - 1; i >= 0; i-- {
		val |= int64(a[i]) & rngMask

		if i > 0 {
			val <<= 8
		}
	}
	return uint64(val)
}

func main() {
	c, err := net.Dial("tcp", "ots-sig.donjon-ctf.io:4001")
	check(err)

	cReader := bufio.NewReader(c)

	var leakedBytes []byte
	for i := 0; i < 7; i++ {
		_, err = cReader.ReadString('\n')
		check(err)

		_, err = cReader.Read([]byte("Enter signature: "))
		check(err)

		_, err = c.Write([]byte("\n"))
		check(err)

		privKeyLine, err := cReader.ReadString('\n')
		check(err)

		_, err = cReader.ReadByte()
		check(err)

		decoded, err := base64.StdEncoding.DecodeString(getPrivKey(privKeyLine))
		check(err)

		leakedBytes = append(leakedBytes, decoded...)
	}

	myRand := rand.New(rand.NewSource(0))
	mySource := myRand.S64.(*rand.RngSource)

	for i := 0; i < len(leakedBytes); i += 7 {
		leak := leakedBytes[i : i+7]

		mySource.Uint64()
		mySource.Vec[mySource.Feed] = int64(restoreUint64(leak))
	}

	ots := wots.NewScheme(sha256.New, myRand)
	priv, pub, _ := ots.GenerateKeyPair()

	pubKeyLine, err := cReader.ReadString('\n')
	check(err)

	pubKey := getPubKey(pubKeyLine)
	myPub := base64.StdEncoding.EncodeToString(pub)

	log.Println("Having:", myPub)
	log.Println("Given :", pubKey)

	if myPub == pubKey {
		var message = []byte("Sign me if you can")

		log.Println("Looks fine! Let's sign!")

		sig, err := ots.Sign(priv, message)
		check(err)

		sigBase := base64.StdEncoding.EncodeToString(sig)

		_, err = c.Write([]byte(sigBase + "\n"))
		check(err)

		log.Println("Signature has been sent:", sigBase)

		for {
			status, err := cReader.ReadString('\n')
			check(err)

			log.Print(status)
		}
	}

	return
}
```

After executing:

```bash
$ go run solve.go
2020/XX/XX 21:58:57 Having: [PUB-BASE64]
2020/XX/XX 21:58:57 Given : [PUB-BASE64]
2020/XX/XX 21:58:57 Looks fine! Lets sign!
2020/XX/XX 21:58:57 Signature has been sent: [SIG-BASE64]
2020/XX/XX 21:58:57 Enter signature: Congratulations! Flag: CTF{m4th_RanD_1s_s0_pr3d1cT4bl3}
2020/XX/XX 21:58:57 Public key: [PUB2-BASE64]
```

If you've read through the PDF above about attacking PRNGs you have read that he describes `/dev/urandom` as cryptographically secure PRNG. In his attack example he uses a value from there to seed his non-cryptographically secure PRNG. We now understand why using secure seeds doesn't solve the problem with deterministic PRNGs.

If you google for `why isn't crypto/rand the default golang` you will find this issue on GitHub: https://github.com/golang/go/issues/11871

```
There have already been incidents where people didn't realize that math/rand was deterministic by default (example),
and even in security-related applications (example).
Additionally, tutorials tend to forego mentioning this potentially-catastrophic default (example).

To resolve this, I propose that the top-level math/rand functions be seeded by crypto/rand's Reader by default.
```

Someone commented this:

`I think that seeding math/rand from crypto/rand will make the problem even worse, because then one could make an (ill-advised) argument for its security.`
