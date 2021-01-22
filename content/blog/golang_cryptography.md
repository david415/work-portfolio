+++
categories = ["blog"]
comments = false
date = "2021-01-13"
slug = ""
tags = ["blog",]
title = "cryptocomrade: Cryptography in Golang"
showpagemeta = false
+++

In this blog post I will share with you how I write cryptography software in Go.
I won't be discussing TLS here at all except to say that the only acceptable
use for TLS is to talk to legacy systems such as "the web". If you are building
something new then the Noise Protocol Framework is a **much** better choice.

This is a work in progress. I'll periodically update this so as to fix mistakes and
add more content.

The cryptocomrade git repo is meant to accompany this blog post by providing code samples
with a CC0 license:

https://github.com/david415/cryptocomrade

However I only added a few simple hash examples so far. I plan to add more soon.


Preliminaries
=============

Cryptography applications should almost always make use of
**Memguard** and **Subtle**.  We can to some extent prevent
[cold boot attacks](https://www.usenix.org/legacy/event/sec08/tech/full_papers/halderman/halderman.pdf)
by using Memguard. By using Subtle constant time comparison functions
you can prevent timing side channels from leaking secret information.

If you are filling up a buffer with random bytes you should do so using
"crypto/rand" and NOT "math/rand". If this operation fails just panic
because if the device ran out of entropy no cryptography can be safely used.

That having been said, sometimes when we write crypto code we choose
to ignore errors where appropriate to do so like in the case where the
io.Read interface semantics don't exactly line up with the API being used.
What I mean is there are cases where a crypto api like a stream
cipher will ALWAYS return successfully from an io.Read
operation. Therefore to ignore io.Read errors in such cases is
acceptable and almost always makes your API cleaner because your
function signatures will be shorter when not having to return an error.


Subtle
------

Use the "crypto/subtle" package:

https://golang.org/pkg/crypto/subtle/

Here's an example of the constant time byte slice comparison:

```go
    if subtle.ConstantTimeCompare(x, y) == 0 {
        return errors.New("values not equal")
    }
```

And here's an [example in the wild](https://github.com/katzenpost/core/blob/master/sphinx/sphinx.go#L276) written by [Yawning Angel](https://github.com/Yawning):
```go
	if subtle.ConstantTimeCompare(pkt[macOff:macOff+crypto.MACLength], mac) != 1 {
		return nil, replayTag[:], nil, errors.New("sphinx: invalid packet, MAC mismatch")
	}
```

In this case Yawning is using it to compare two MACs, a good use case for the **"crypto/subtle"** module
and this is exactly why the "crypto/hmac" module has a wrapper function for "subtle.ConstantTimeCompare" called
"Equal" https://golang.org/pkg/crypto/hmac/#Equal which returns a bool.
Use either one but pick one and use it to avoid leaking timing information.

Memguard
--------
**MemGuard: Software enclave for storage of sensitive information in memory.**

See https://github.com/awnumar/memguard for the complete feature set.

In the author's words:
```text
This package attempts to reduce the likelihood of sensitive data being
exposed when in memory. It aims to support all major operating systems
and is written in pure Go.
```

Memguard has thorough [documentation](https://godoc.org/github.com/awnumar/memguard)
and plenty of [examples](https://github.com/awnumar/memguard/tree/master/examples), however,
here's an extensive example found in the wild:

https://github.com/katzenpost/doubleratchet/blob/master/ratchet.go

It's certainly possible to use Memguard and not get any benefit from it. This can happen if you
are making superfluous copies of your key material. Such copies essentially defeat the purpose of
using Memguard. Instead you are suppose to pass around references to Memguard's opaque types
and use those to guard the key material.

Cryptographic Hashing
=====================

[Use Blake2 because it's harder, better, faster, stronger](https://leastauthority.com/blog/blake2-harder-better-faster-stronger-than-md5/).

The [Blake2 website](https://www.blake2.net/) says:
```text
BLAKE2b is more efficient on 64-bit CPUs and BLAKE2s is more efficient on
8-bit, 16-bit, or 32-bit CPUs
```

So it seems to me that Blake2b is good enough for most use cases since many mobile devices
these days have 64bit CPUs. Blake2 is very versitile and has multiple hash output lengths,
can be used as a KDF with it's XOF (extensible output function) or a MAC (message
authentication code) with it's built-in keying mechanism. Blake2 is
not vulnerable to the [length extension attack](https://en.wikipedia.org/wiki/Length_extension_attack)
so an HMAC construction is not needed.

The [rfc7693](https://tools.ietf.org/html/rfc7693) states that:

```text
Both BLAKE2b and BLAKE2s are believed to be highly secure and perform
well on any platform, software, or hardware. BLAKE2 does not require
a special "HMAC" (Hashed Message Authentication Code) construction
for keyed message authentication as it has a built-in keying
mechanism.
```

I suggest using the Blake2b implementation from the supplementary Go cryptography libraries:

https://golang.org/x/crypto/blake2b

Hash a byte slice:

```Go
// SimplexHash is the simplest usage of a hash function
// and is suitable if you only have one use case.
func SimplexHash(data []byte) []byte {
	out := blake2b.Sum256(data)
	return out[:]
}
```

Instead you probably want to use something like this for multiple
use cases:

```Go
// Hash hashes the data with contextInfo used as the key
// for blake2's keying mechanism. contextInfo is meant to
// be a human readable string that ensures each use case of
// the hash function will have different outputs.
func Hash(contextInfo string, data []byte) ([]byte, error) {
	if len(contextInfo) > blake2b.Size {
		return nil, errors.New("contextInfo size error")
	}
	h, err := blake2b.New256([]byte(contextInfo))
	if err != nil {
		return nil, err
	}
	_, err = h.Write(data)
	if err != nil {
		return nil, err
	}
	return h.Sum(nil), nil
}
```

Key Derivation Function
=======================

Just use [HKDF](https://eprint.iacr.org/2010/264.pdf).
Specifically, HKDF-SHA256 is probably sufficient for most use
cases when your initial key material is 32 bytes in length.
You MUST only feed your KDF a key which has uniform entropy. If
you need to derive keys from a password or passphrase then you should
first use a "key stretching function" which I discuss in the **Hashing
Passwords** section below.

Remember how I mentioned above that Blake2 is versatile and can be
used as a KDF and a MAC? Well, I suggest we NOT using Blake2 for all
of these use cases in a single application because if you do and it
becomes weakened or compromised by a cryptographic attack then several
parts of your construction will be broken instead of just one.

(But if you insist on using Blake2b as a KDF this is
how you'd do it: **here's an example: blah blah url goes here**)

I suggest using the HKDF implementation from the supplementary Go cryptography libraries:

https://golang.org/x/crypto/hkdf

Here's a simple example usage:

```Go
// KDF returns a slice of derive keysNum number of derived keys of size keySize.
// The given secret must be a uniform entropy secret (ie not a password) and
// info is an optional non-secret which may be omitted.
//
// Note: in practice you probably don't want to use this particular function
// but instead use the HKDF directly.
func KDF(secret, salt, info []byte, keysNum, keySize int) ([][]byte, error) {
	if len(salt) != hash().Size() {
		return nil, errors.New("wrong salt size")
	}
	hash := sha256.New
	hkdf := hkdf.New(hash, secret, salt, info)
	var keys [][]byte
	for i := 0; i < keysNum; i++ {
		key := make([]byte, keySize)
		if _, err := io.ReadFull(hkdf, key); err != nil {
			panic(err)
		}
		keys = append(keys, key)
	}
	return keys, nil
}
```

The **hkdf type** implements the **io.Reader** interface so you can read from it with any
size byte slice that you'd like. Usually I use it to generate 32 byte keys.


Hashing Passwords
=================

Use Argon2 because it won the [Password Hashing Competition](https://www.password-hashing.net/).
It's suitable to use if you are storing password hashes in a database or if you have to derive
cryptographic keys from a password or passphrase.

I suggest using the Argon2id implementation from the supplementary Go cryptography libraries:

https://golang.org/x/crypto/argon2

```text
Argon2id is side-channel resistant and provides better brute-
force cost savings due to time-memory tradeoffs than Argon2i. 
```

This example consumes 64MB of memory. So depending on your device
and your application you'll want to adjust this parameter:

```Go
// HashPassword returns a 32 byte cryptographic key given
// a password and an entropic salt.
func HashPassword(password, salt []byte) []byte {
	return argon2.IDKey(password, salt, 1, 64*1024, 4, 32)
}
```

Note that the last parameter is key size, thus you won't need to use
Argon2 with HKDF because you can specify any key size you'd
like. However the previously mentioned HKDF uses the io.Reader
interface which makes it quite easy to derive multiple keys without
having to fiddle with byte slices. It wouldn't hurt to process the
output of Argon2 with HKDF and the performance hit is probably
negligible.


Message Authentication Code
===========================

First choice: Just use NaCl auth because it exists and it's easy to use.
NaCl auth uses HMAC-SHA512:

https://pkg.go.dev/golang.org/x/crypto/nacl/auth


Otherwise use Blake2b's keyed hash feature for a MAC:

```Go
// Blake2bMAC is a MAC that uses Blake2b's keyed hash mechanism
// instead of an HMAC construction. The output is of size MACSize.
func Blake2bMAC(key, data []byte) []byte {
	h, err := blake2b.New(MACSize, key)
	if err != nil {
		panic(err)
	}
	h.Write(data)
	return h.Sum(nil)
}

// ValidBlake2bMAC reports whether Blake2b messageMAC is a valid MAC tag for message.
func ValidBlake2bMAC(message, messageMAC, key []byte) bool {
	expectedMAC := Blake2bMAC(key, message)
	return hmac.Equal(messageMAC, expectedMAC)
}
```

If you need the MAC tag to be 32 bytes long are already using Blake2b
as a hash then you should select a different MAC such as
**HMAC-SHA256**.

```Go
package mac

import (
	"crypto/hmac"
	"crypto/sha256"
)

// HMACSHA256 returns the HMAC-SHA256 authentication code for the given message and key.
func HMACSHA256(message, key []byte) []byte {
	mac := hmac.New(sha256.New, key)
	mac.Write(message)
	return mac.Sum(nil)
}

// ValidMAC reports whether messageMAC is a valid HMAC tag for message.
func ValidHMACSHA256(message, messageMAC, key []byte) bool {
	return hmac.Equal(messageMAC, HMACSHA256(message, key))
}
```


AEAD Ciphers
============

AEAD stands for Authenticated Encryption with Associated Data.  Unless
the AEAD has a synthetic IV, nonce reuse is not safe. The way this
situation was handled in the past was to turn the nonce into a counter
and always increment the nonce with each use. This turns out to be
very difficult to do properly in practice. So the preferred solution
is instead to use bigger nonces that can safely be randomly
generated. That is, the nonce size should be big enough that randomly
generating it has an extremely low probability of a nonce being
reused.

But I would instead recommend the AEAD known as XSalsa20Poly1305 or
XChacha20Poly1305 which is composed of the Poly1305 MAC and the
XSalsa20 or the XChacha20 stream cipher.  Salsa20 was the original but
with it's 8 byte nonce it required careful nonce management. The X
variant, XSalsa20 increased the nonce size to 24 bytes. According to
the [libsodium docs](https://libsodium.gitbook.io/doc/advanced/stream_ciphers/xsalsa20):

```text
But with XSalsa20's longer nonce, it is safe to generate nonces using
randombytes_buf() for every message encrypted with the same key without
having to worry about a collision.
```

I'm not clear on what the differences are between XSalsa20 and
XChacha20 except that initially Salsa20 had a security proof and
Chacha20 did not because of being newer. Whereas they now both have
security proofs and so do their bigger nonce variants.

The older recommendation would be to use the AES-GCM AEAD but I think we're
past that phase in cryptography history now (although NIST lags behind of course).
[In agl's words](https://www.imperialviolet.org/2013/10/07/chacha20.html):

```text
These are primitives developed by Dan Bernstein and are fast,
secure, have high quality, public domain implementations, are
naturally constant time and have nearly perfect key agility.
```

Use [NaCl secretbox](https://nacl.cr.yp.to/secretbox.html) to
symmetrically encrypt small messages (small enough to fit in memory).
NaCl secretbox uses XSalsa20 and Poly1305. I really like the NaCl
secretbox API for golang, it's very simple:

https://pkg.go.dev/golang.org/x/crypto/nacl/secretbox

Another good choice would be to use XChacha20Poly1305 AEAD cipher directly:

https://pkg.go.dev/golang.org/x/crypto/chacha20poly1305

If your encryption payload is too big to fit in memory then you could use the "age" API.
See the next section "Encrypting Files".

Asymmetric Authenticated Encryption
-----------------------------------

Use [NaCl box](https://nacl.cr.yp.to/box.html) box to asymmetrically encrypt small messages.
Box uses Curve25519, XSalsa20 and Poly1305.

https://pkg.go.dev/golang.org/x/crypto/nacl/box


Encrypting Files
================

*This section is also known as the **Encrypting Bigass Files** section.*

If you have some bigass files to encrypt then using an AEAD cipher
directly without a buffering layer will result in loading the
entire plaintext into memory all at once. Not good if your file is
bigger than your device's available memory. Instead just use **age**
because it exists and is easy to use. Although **age** is a
commandline tool it's also a cleanly written Go encryption library.

Age has support for encrypting using symmetric and asymmetric ciphers.

Specification:
https://docs.google.com/document/d/11yHom20CrsuX8KQJXBBw04s80Unjv8zCg_A7sPAX_9Y/preview

Source code:
https://github.com/FiloSottile/age/

Golang API Docs:
https://pkg.go.dev/filippo.io/age


Cryptographic Transports
========================

TODO: discuss the [Noise Protocol Framework](https://noiseprotocol.org/).

Here's an example of Noise being used with the New Hope Simple hybrid forward secret handshake modifier, it's full handshake protocol descriptor string is: **Noise_XXhfs_25519+NewHopeSimple_ChaChaPoly_Blake2b**.

https://github.com/katzenpost/core/blob/master/wire/session.go

Of course the next step is to make Kyber work with Noise. I haven't gotten around to it yet although [it's been done in Rust already](https://github.com/mcginty/snow/issues/39).

Summary
=======

* Hash: Blake2b256
* KDF: HKDF-SHA256
* Password Hashing: Argon2
* MAC: Blake2b, HMAC-SHA256
* Encrypting Bigass Files: "age"
* Symmetric Encryption: NaCl Secretbox or XChaCha20-Poly1305 AEAD
* Asymmetric Encryption: NaCl Box
* Transport Encryption: Noise w/ XX handshake pattern
