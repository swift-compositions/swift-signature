# swift-signature

Use this package when you hold an RSA private key and need an RS256 signature over some bytes, for example to sign a JSON Web Token such as a GitHub App assertion. Start from `Signature.RSA.Key`, built from a PEM string or DER bytes, and pass it with the message to `Signature.RS256.sign(message:key:)`.

## Laws

- RS256 here is RSASSA-PKCS1-v1_5 over SHA-256. The message is hashed with SHA-256 from swift-fips-180-4, and only the digest goes to the platform key facility. The result verifies as a message-level RS256 signature.
- Signing needs Apple's Security framework. Where it can't be imported, `Signature.RS256.sign(message:key:)` always throws `Signature.RS256.Error.unsupportedPlatform`.
- A key isn't checked when you create it. `Signature.RSA.Key.init(der:)` takes any bytes, and a key the platform can't import fails only at signing, with `Signature.RS256.Error.malformedKey`.
- `Signature.RSA.Key.init(pem:)` reads the base64 between the `-----BEGIN` line and the next `-----END` line, ignoring blank lines and surrounding whitespace. A header that names `RSA PRIVATE KEY` is PKCS#1 and its bytes are kept as they are. Any other header is read as PKCS#8 and unwrapped to the inner key. A missing header, an empty body or bad base64 throws `Signature.RSA.Key.Error.malformed`.
- `Signature.RSA.Key.unwrap(_:)` takes the private key octet string out of a PKCS#8 wrapper without checking the algorithm it names. DER that is truncated or has an unusable length, including an indefinite length or one longer than four bytes, throws `Signature.RSA.Key.Error.malformed` and never traps.
