Flexible whitelist/blacklist testing

### Synopsis

```js
var GreyList = require('greylist');
var greylist = new GreyList(options);
var passed = greylist.test(string);
```

### Options

The options object may itself be undefined. When it is defined, there are two options:

#### `options.white`
A whitelist. Only listed strings can pass. If `undefined`, all strings are included. If an empty array, all strings are excluded.

#### `options.black`
A blacklist. Listed strings are blocked. If `undefined` or an empty array, all strings are included.

#### Whitelist and blacklist values
May be any of:
* an array — elements that are not already strings or `RegExp` objects are converted to strings
* a `RegExp` — converted to a single-element array
* a string — converted to a single-element array
* an object — converted to array comprised of the keys of the object's enumerable members that have defined values
* `undefined` — a no-op

### Methods

#### `test`
The single defined method, `test(string)`, returns a boolean subject to the following conditions:
1. If `whitelist` is defined, the test string must [`match`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String/match) one of its elements.
2. If `blacklist` is defined, the test string must not match any of its elements.

This method is a suitable argument to [`Array.prototype.filter`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) so long as it's properly bound:
```js
var words = [...]; // provide a list of words to exclude
var greylist = new GreyList(options);

// The following three lines are equivalent:
var acceptableWords = words.filter(greylist.test, greylist);
var acceptableWords = words.filter(function(word) { return greylist.test(word); });
var byGreylist = greylist.test.bind(greylist), acceptableWords = words.filter(byGreylist);
```

### Examples

```js
var greylist = new GreyList(['abc','de','f']);
greylist.test('abc') // true: whitelisted and no blacklist
greylist.test('ab') // false: not in whitelist
greylist.test('de') // true: whitelisted and no blacklist
greylist.test('f') // true: whitelisted and no blacklist
greylist.test('g') // false: not in whitelist

var threeOrMore = /^[a-z]{3,}$/;
var greylist = new GreyList(undefined, [threeOrMore, 'f']);
greylist.test('abc') // false: no whitelist but strings of length 3 or more are blacklisted
greylist.test('ab') // true: no whitelist and not blacklisted
greylist.test('de') // true: no whitelist and not blacklisted
greylist.test('f') // false: no whitelist but blacklisted
greylist.test('g') // true: no whitelist and not blacklisted

var greylist = new GreyList(['abc','de','f'], [threeOrMore, 'f']);
greylist.test('abc') // false: whitelisted but strings of length 3 or more are blacklisted
greylist.test('ab') // false: not in whitelist
greylist.test('de') // true: whitelisted and not blacklisted
greylist.test('f') // false: whitelisted but blacklisted
greylist.test('g') // false: not in whitelist
```