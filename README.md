@fibjs/keygrip
=======

[![NPM version][npm-image]][npm-url]
[![build status][travis-image]][travis-url]
[![appveyor build status][appveyor-image]][appveyor-url]
[![Test coverage][codecov-image]][codecov-url]
[![David deps][david-image]][david-url]
[![Known Vulnerabilities][snyk-image]][snyk-url]
[![npm download][download-image]][download-url]

[npm-image]: https://img.shields.io/npm/v/@fibjs/keygrip.svg?style=flat-square
[npm-url]: https://npmjs.org/package/@fibjs/keygrip
[travis-image]: https://img.shields.io/travis/fibjs-modules/keygrip.svg?style=flat-square
[travis-url]: https://travis-ci.org/fibjs-modules/keygrip
[appveyor-image]: https://ci.appveyor.com/api/projects/status/9nv7t7tjl4l7i25b?svg=true
[appveyor-url]: https://ci.appveyor.com/project/ngot/keygrip
[codecov-image]: https://img.shields.io/codecov/c/github/fibjs-modules/keygrip.svg?style=flat-square
[codecov-url]: https://codecov.io/github/fibjs-modules/keygrip?branch=master
[david-image]: https://img.shields.io/david/fibjs-modules/keygrip.svg?style=flat-square
[david-url]: https://david-dm.org/fibjs-modules/keygrip
[snyk-image]: https://snyk.io/test/npm/@fibjs/keygrip/badge.svg?style=flat-square
[snyk-url]: https://snyk.io/test/npm/@fibjs/keygrip
[download-image]: https://img.shields.io/npm/dm/@fibjs/keygrip.svg?style=flat-square
[download-url]: https://npmjs.org/package/@fibjs/keygrip

Keygrip is a [fibjs](http://fibjs.org/) module for signing and verifying data (such as cookies or URLs) through a rotating credential system, in which new server keys can be added and old ones removed regularly, without invalidating client credentials.

## Install

    $ npm install @fibjs/keygrip

## API

### keys = new Keygrip([keylist], [hmacAlgorithm], [encoding])

This creates a new Keygrip based on the provided keylist, an array of secret keys used for SHA1 HMAC digests. `keylist` is obligatory. `hmacAlgorithm` defaults to `'sha1'` and `encoding` defaults to `'base64'`.

Note that the `new` operator is also optional, so all of the following will work when `Keygrip = require("keygrip")`:

```javascript
keys = new Keygrip(["SEKRIT2", "SEKRIT1"])
keys = Keygrip(["SEKRIT2", "SEKRIT1"])
keys = require("keygrip")()
keys = Keygrip(["SEKRIT2", "SEKRIT1"], 'sha256', 'hex')
keys = Keygrip(["SEKRIT2", "SEKRIT1"], 'sha256')
keys = Keygrip(["SEKRIT2", "SEKRIT1"], undefined, 'hex')
```

The keylist is an array of all valid keys for signing, in descending order of freshness; new keys should be `unshift`ed into the array and old keys should be `pop`ped.

The tradeoff here is that adding more keys to the keylist allows for more granular freshness for key validation, at the cost of a more expensive worst-case scenario for old or invalid hashes.

Keygrip keeps a reference to this array to automatically reflect any changes. This reference is stored using a closure to prevent external access.

### keys.sign(data)

This creates a SHA1 HMAC based on the _first_ key in the keylist, and outputs it as a 27-byte url-safe base64 digest (base64 without padding, replacing `+` with `-` and `/` with `_`).

### keys.index(data, digest)

This loops through all of the keys currently in the keylist until the digest of the current key matches the given digest, at which point the current index is returned. If no key is matched, `-1` is returned.

The idea is that if the index returned is greater than `0`, the data should be re-signed to prevent premature credential invalidation, and enable better performance for subsequent challenges.

### keys.verify(data, digest)

This uses `index` to return `true` if the digest matches any existing keys, and `false` otherwise.

## Example

```javascript
// ./test.js
var assert = require("assert")
  , Keygrip = require("keygrip")
  , keylist, keys, hash, index

// but we're going to use our list.
// (note that the 'new' operator is optional)
keylist = ["SEKRIT3", "SEKRIT2", "SEKRIT1"]
keys = Keygrip(keylist)
// .sign returns the hash for the first key
// all hashes are SHA1 HMACs in url-safe base64
hash = keys.sign("bieberschnitzel")
assert.ok(/^[\w\-]{27}$/.test(hash))

// .index returns the index of the first matching key
index = keys.index("bieberschnitzel", hash)
assert.equal(index, 0)

// .verify returns the a boolean indicating a matched key
matched = keys.verify("bieberschnitzel", hash)
assert.ok(matched)

index = keys.index("bieberschnitzel", "o_O")
assert.equal(index, -1)

// rotate a new key in, and an old key out
keylist.unshift("SEKRIT4")
keylist.pop()

// if index > 0, it's time to re-sign
index = keys.index("bieberschnitzel", hash)
assert.equal(index, 1)
hash = keys.sign("bieberschnitzel")
```

## TODO

* Write a library for URL signing

Copyright
---------

Copyright (c) 2012 Jed Schmidt. See LICENSE.txt for details.

Send any questions or comments [here](http://twitter.com/jedschmidt).
