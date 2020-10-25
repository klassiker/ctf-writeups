# FluxCloud Serverless

## Task

Categories: web

Difficulty: baby

To host stuff like our website, we developed our own cloud because we do not trust the big evil corporations! Of course we use cutting edge technologies, like serverless. Since we know what we are doing, it is totally unhackable. If you want to try, you can check out the demo and if you can access the secret, you will even get a reward :)

Note: This version of the challenge contains a bypass that has been fixed in FluxCloud Serverless 2.0.

https://serverless.cloud.flu.xxx

File: fluxcloud-serverless.zip

## Solution

We get a nodejs application that is deployed using docker.

It serves the static files and handles requests to `/demo` which is forwarded to `serverless`.

`serverless` waits for this POST request and creates a new instance for it, redirects the user to it.

For the `:deploymentId` the `deploymentRouter` is used. The request is first passed into the `waf` and then to the `app`.

The `app` provides a `/flag` request which sends the content of the `FLAG` variable.

If we try to request this, the `waf` will block us.

It filters out bad strings and decodes url-encoding:

```javascript
const badStrings = [
    'flag',
];

function checkRecursive(value) {
    // don't get bypassed by double-encoding
    const hasPercentEncoding = /%[a-fA-F0-9]{2}/.test(value);
    if (hasPercentEncoding) {
        return checkRecursive(decodeURIComponent(value));
    }

    // check for any bad word
    for (const badWord of badStrings) {
        if (value.includes(badWord)) {
            return true;
        }
    }
    return false;
}
```

There are more strings in there but I filtered them out.

First I tried unicode character normalization:

```python
>>> import unicodedata
>>> [[i, chr(i), unicodedata.normalize("NFKD", chr(i))] for i in range(100000) if unicodedata.normalize("NFKC", chr(i)) in [chr(i) for i in range(97,123)]]
```

Trying any of the non-ascii results: no luck.

But whait, what about uppercase characters? Keep it simple, they said.

A request for `/demo/:deploymentId/flaG` was successfull.
