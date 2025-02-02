<img align="right" src="http://media.taskcluster.net/logo/logo-96x120.png" />

# slugid - Compressed UUIDs for Node.js

A node.js module for generating v4 UUIDs and encoding them into 22 character
URL-safe base64 slug representation (see [RFC 4648 sec.
5](http://tools.ietf.org/html/rfc4648#section-5)).

Slugs are url-safe base64 encoded v4 uuids, stripped of base64 `=` padding.
They are generated with the [uuid package](https://www.npmjs.com/package/uuid) which
is careful to use a cryptographically strong random number generator on platforms
that make one available.

There are two methods for generating slugs - `slugid.v4()` and `slugid.nice()`.

The `slugid.v4()` method returns a slug from a randomly generated v4 uuid. The
`slugid.nice()` method returns a v4 slug which conforms to a set of "nice"
properties. At the moment the only "nice" property is that the slug starts with
[A-Za-f], which in turn implies that the first (most significant) but of its
associated uuid is set to 0.

The purpose of the `slugid.nice()` method is to support having slugids which
can be used in more contexts safely. Regular slugids can safely be used in
urls, and for example in AMQP routing keys. However, slugs beginning with `-`
may cause problems when used as command line parameters.

In contrast, slugids generated by the `slugid.nice()` method can safely be used
as command line parameters. This comes at a cost to entropy (121 bits vs 122
bits for regular v4 slugs).

Slug consumers should consider carefully which of these two slug generation
methods to call. Is it more important to have maximum entropy, or to have
slugids that do not need special treatment when used as command line
parameters? This is especially important if you are providing a service which
supplies slugs to unexpecting tool developers downstream, who may not realise
the risks of using your regular v4 slugs as command line parameters, especially
since this would arise only as an intermittent issue (one time in 64).

Generated slugs take the form `[A-Za-z0-9_-]{22}`, or more precisely:

* `slugid.v4()` slugs conform to
  `[A-Za-z0-9_-]{8}[Q-T][A-Za-z0-9_-][CGKOSWaeimquy26-][A-Za-z0-9_-]{10}[AQgw]`

* `slugid.nice()` slugs conform to
  `[A-Za-f][A-Za-z0-9_-]{7}[Q-T][A-Za-z0-9_-][CGKOSWaeimquy26-][A-Za-z0-9_-]{10}[AQgw]`

RFC 4122 defines the setting of 6 bits of the v4 UUID which determines these
regular expressions.

```js
var slugid = require('slugid');

// Generate "nice" URL-safe base64 encoded UUID version 4 (random)
var slug = slugid.nice(); // a8_YezW8T7e1jLxG7evy-A
```

Encode / Decode
---------------
```js
var slugid = require('slugid');

// Generate URL-safe base64 encoded UUID version 4 (random)
var slug = slugid.v4();

// Get UUID on the form xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx
var uuid = slugid.decode(slug);

// Compress to slug again
assert(slug == slugid.encode(uuid));
```

License
-------
The `slugid` library is released on the MIT license, see the `LICENSE` for
complete license.
