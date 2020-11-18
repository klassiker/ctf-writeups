# One Time-Based Signature

## Task

We used a quantum-resistant signature algorithm. Will you be able to break it? Show us you can sign a message without knowing our private key!

Url: ots-sig.donjon-ctf.io:4000

File: challenge.go

## Solution

Connecting to the address:

```bash
$ nc ots-sig.donjon-ctf.io 4000
Public key: [REDACTED]
Enter signature: ^C
```

In the source see the import of `github.com/dchest/wots` and the following code:

```go
package main

import (
	"bufio"
	"crypto/sha256"
	"encoding/base64"
	"fmt"
	"io/ioutil"
	"math/rand"
	"os"
	"strings"
	"time"

	"github.com/dchest/wots"
)

const defaultFlag string = "CTF{xxx}"

func main() {
	var message = []byte("Sign me if you can")

	var ots = wots.NewScheme(sha256.New, rand.New(rand.NewSource(time.Now().UnixNano()/1e6)))
	_, pub, _ := ots.GenerateKeyPair()

	fmt.Println("Public key:", base64.StdEncoding.EncodeToString(pub))

	reader := bufio.NewReader(os.Stdin)
	fmt.Print("Enter signature: ")
	text, err := reader.ReadString('\n')
	if err != nil {
		fmt.Println("Error occurred. Please try again later.")
		return
	}

	text = strings.TrimSuffix(text, "\n")
	signature, _ := base64.StdEncoding.DecodeString(text)
	if ots.Verify(pub, message, signature) {
		fmt.Print("Congratulations! Flag: ")

		flag, err := ioutil.ReadFile("secret")
		if err != nil {
			fmt.Println(defaultFlag)
		} else {
			fmt.Println(string(flag))
		}
	} else {
		fmt.Println("Try again.")
	}
}
```

We initialize the new scheme with our own random which is based on the current time. This is never a good approach. To make things even worse we throw away the last 6 digits by diving through `1e6`.

This is why the challenge is called `Time-Based Signature`.

Let's get our system clock synchronized and start attacking! `time.Now()` returns the current local time so we first we start searching with more possibilities to find the correct millisecond offset for a provided public key:

But what about timezones? Do I have to check for the right one? No. Unix time is calculated since Epoch.

```go
func main() {
	local := time.Now().UnixNano()/1e6
	conn, err := net.Dial("tcp", "ots-sig.donjon-ctf.io:4000")
	check(err)

	connReader := bufio.NewReader(conn)
	status, err := connReader.ReadString('\n')
	check(err)

	needBase := strings.TrimSuffix(strings.TrimPrefix(status, "Public key: "), "\n")

	var found bool
	var i int64
	for i = -1000; i < 1000 && !found; i++ {
		ots := wots.NewScheme(sha256.New, rand.New(rand.NewSource(local + i)))
		_, pub, _ := ots.GenerateKeyPair()
		pubBase := base64.StdEncoding.EncodeToString(pub)

		if pubBase == needBase {
			found = true

			log.Println("Found time offset i:", i)
		}
	}
}
```

If your clock is in good sync then `i` will be greater or equal to `0` and probably less than `100`.

Now we just need to extend our code to save the generated private key and used scheme and sign the required message:

```go
func main() {
  ...
  if !found {
		log.Println("Failed finding time offset! Try synchronizing your clock!")
		return
	}

	message := []byte("Sign me if you can")

	sig, err := ots.Sign(privateKey, message)
	check(err)

	sigBase := base64.StdEncoding.EncodeToString(sig)
	_, err = conn.Write([]byte(sigBase + "\n"))
	check(err)

	log.Println("Signature has been sent:", sigBase)

	for {
		status, err = connReader.ReadString('\n')
		check(err)

		log.Print(status)
	}
}
```

Running it:

```bash
$ go run solve.go
2020/XX/XX 17:09:37 PublicKey to find: [REDACTED]
2020/XX/XX 17:09:37 Found time offset: 79
2020/XX/XX 17:09:37 Signature has been sent: [REDACTED]
2020/XX/XX 17:09:37 Enter signature: Congratulations! Flag: CTF{e4sY_brUt3f0Rc3}
2020/XX/XX 17:09:37 EOF
```
