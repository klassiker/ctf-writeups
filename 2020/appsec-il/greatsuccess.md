# GreatSuccess

## Task

Great React! I'm very excite!

File: memeApp.ipa

## Solution

We start off by extracting the zip archive:

```bash
$ unzip memeApp.ipa
```

We now can start digging around the application. Things of interest:

 - `memeApp` - `Mach-O universal binary with 2 architectures: [armv7:Mach-O armv7 executable, flags:<NOUNDEFS|DYLDLINK|TWOLEVEL|WEAK_DEFINES|BINDS_TO_WEAK|PIE>] [arm64]`
 - `main.jsbundle` - `ASCII text, with very long lines`
 - `AccessibilityResources.bundle/en.lproj/Localizable.strings` - `ASCII text`
 - `assets/app.json` - `JSON data`

The last two don't contain anything interesting. Before we start with the binary, let's look at the jsbundle.

Fast scrolling through yields this:

 - `SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED`
 - `FxUCKisKTiwHHjRVFi8GHgtFQEoJFhcXKjYOBQRFnMcIgAfAFVtYCIEBhw9NhcKFAgYRywTBAQTWFceMAweFGA2MwkuBApJZRMKBAhUbVI4AS0KKww8CDI6H0UlIUc0LFdtWiI6HBY6NhAQKAYcXzpgNiI6RltfOjoQHBEMGwAoEA1JZzE=`
 - `https://greatsuccess.appsecil.ctf.today/key` -> `VeryNiceKey,ILike123`

The second one is base64 but the decoded version is useless. We need to find out what's going on here.

This might be worth reading: https://github.com/OWASP/owasp-mstg/blob/master/Document/0x06c-Reverse-Engineering-and-Tampering.md#patching-react-native-applications

I started of beautifying the jsbundle, `JStillery` threw me errors but there are lots of other beautifiers out there.

I then opened it in an IDE that analyzes the script and supports me with `Find Usage` and `Go to declaration` tools. After all, I didn't really need them but it is a good starting point.

We find our URL again:

```javascript
__d(function(g, r, i, a, m, e, d) {
    var t = r(d[0]),
        c = r(d[1]);
    Object.defineProperty(e, "__esModule", {
        value: !0
    }), e.default = void 0;
    var n = c(r(d[2])),
        l = t(r(d[3])),
        f = r(d[4]),
        o = r(d[5]),
        u = {
            uri: "[not useful]"
        };

    function s(t) {
        var c = (0, l.useState)("Get Memed"),
            f = (0, n.default)(c, 2),
            o = f[0],
            u = f[1],
            s = (0, l.useState)(!1),
            h = (0, n.default)(s, 2),
            I = h[0],
            w = h[1];
        return (0, l.useEffect)(function() {
            fetch(t).then(function(t) {
                return t.text()
            }).then(function(t) {
                console.log('response: ' + t), u(t)
            }).catch(function() {
                w(!0)
            })
        }, [t]), [o, I]
    }
    var h = o.StyleSheet.create({...stuff...}),
        I = function() {
            var t = s("https://greatsuccess.appsecil.ctf.today/key"),
                c = (0, n.default)(t, 2),
                I = c[0],
                w = (c[1], (0, f.decrypt)(I, "FxUCKisKTiwHHjRVFi8GHgtFQEoJFhcXKjYOABQRFnMcIgAfAFVtYCIEBhw9NhcKFAgYRywTBAQTWFceMAweFGA2MwkuBApJZRMKBAhUbVI4AS0KKww8CDI6H0UlIUc0LFdtWiI6HBY6NhAQKAYcXzpgNiI6RltfOjoQHBEMGwAoEA1JZzE="));
            return console.log(w), l.default.createElement(o.View, {
                style: h.container
            }, l.default.createElement(o.ImageBackground, {
                source: u,
                style: h.image
            }, l.default.createElement(o.Text, {
                style: h.text
            }, w)))
        };
    e.default = I
}, 402, [9, 1, 14, 55, 403, 2]);
```

If you've ever looked at bundled javascript it's not that hard to figure out what's going on here.

We have a module resolver function `__d` which provides it's first parameter with the dependencies listed in the third parameter. The position of the module itself is the second parameter.

So we have module `402` that depends on `[9, 1, 14, 55, 403, 2]`. The inner function `s` seems to fetch the content of a page. The fetched content is probably used as a key in the call to decrypt the base64. `f = r(d[4])` is module `403`:

```javascript
__d(function(g, r, i, a, m, e, d) {
    var t = r(d[0]);
    Object.defineProperty(e, "__esModule", {
        value: !0
    }), e.encrypt = function(t, o) {
        return xorEncrypt = function(t, c) {
            return n.default.map(c, function(n, c) {
                return n.charCodeAt(0) ^ t.charCodeAt(Math.floor(c % t.length))
            })
        }, b64Encode = function(t) {
            var n, o, h, u, f, l, p = 0,
                A = '';
            if (!t) return t;
            do {
                n = (f = t[p++] << 16 | t[p++] << 8 | t[p++]) >> 18 & 63, o = f >> 12 & 63, h = f >> 6 & 63, u = 63 & f, A += c.charAt(n) + c.charAt(o) + c.charAt(h) + c.charAt(u)
            } while (p < t.length);
            return ((l = t.length % 3) ? A.slice(0, l - 3) : A) + '==='.slice(l || 3)
        }, o = this.xorEncrypt(t, o), this.b64Encode(o)
    }, e.decrypt = function(t, o) {
        return b64Decode = function(t) {
            var n, o, h, u, f, l, p = 0,
                A = [];
            if (!t) return t;
            t += '';
            do {
                n = (l = c.indexOf(t.charAt(p++)) << 18 | c.indexOf(t.charAt(p++)) << 12 | (u = c.indexOf(t.charAt(p++))) << 6 | (f = c.indexOf(t.charAt(p++)))) >> 16 & 255, o = l >> 8 & 255, h = 255 & l, A.push(n), 64 !== u && (A.push(o), 64 !== f && A.push(h))
            } while (p < t.length);
            return A
        }, xorDecrypt = function(t, c) {
            return n.default.map(c, function(n, c) {
                return String.fromCharCode(n ^ t.charCodeAt(Math.floor(c % t.length)))
            }).join('')
        }, o = this.b64Decode(o), this.xorDecrypt(t, o)
    }, e.b64Table = void 0;
    var n = t(r(d[1])),
        c = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=';
    e.b64Table = c
}, 403, [1, 404]);
```

We see some xor-ing going on. Let's write a function to xor the base64 decoded chars and print them out:

```javascript
// Extracted from above
var decrypt = function(t) {
            var n, o, h, u, f, l, p = 0,
                A = [];
            if (!t) return t;
            t += '';
            do {
                n = (l = c.indexOf(t.charAt(p++)) << 18 | c.indexOf(t.charAt(p++)) << 12 | (u = c.indexOf(t.charAt(p++))) << 6 | (f = c.indexOf(t.charAt(p++)))) >> 16 & 255, o = l >> 8 & 255, h = 255 & l, A.push(n), 64 !== u && (A.push(o), 64 !== f && A.push(h))
            } while (p < t.length);
            return A
}

var c = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=';
var k = "VeryNiceKey,ILike123";

var chars = decrypt("FxUCKisKTiwHHjRVFi8GHgtFQEoJFhcXKjYOABQRFnMcIgAfAFVtYCIEBhw9NhcKFAgYRywTBAQTWFceMAweFGA2MwkuBApJZRMKBAhUbVI4AS0KKww8CDI6H0UlIUc0LFdtWiI6HBY6NhAQKAYcXzpgNiI6RltfOjoQHBEMGwAoEA1JZzE=").map(x => String.fromCharCode(x));

var out = [];
for (var i = 0; i < chars.length; i++) {
  out.push(String.fromCharCode(chars[i].charCodeAt(0) ^ k.charCodeAt(i % k.length)));
}

console.log(out.join(''));
```

We get: `AppSec-IL{My_country_send_me_to_United_States_to_make_movie-film._Please,_come_and_see_my_film._If_it_not_success,_I_will_be_execute.}`
