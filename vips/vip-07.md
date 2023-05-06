<!-- markdownlint-disable MD043 -->

# VIP-07 --- JavaScript `Crypto` Capability

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

`v-language:javascript`

`capability:Crypto`

`required-capability:Async`

---

- [1. Capability Description](#1-capability-description)
- [2. Available Symbols](#2-available-symbols)

---

## 1. Capability Description

The `Crypto` capability makes the functionality provided by the [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API) available.

The `Async` capability is **REQUIRED** for this capability to be available.

## 2. Available Symbols

The following table details what symbols are made available by this capability.
Note the absence of the following symbols:

- **`Crypto.getRandomValues()` and `Crypto.randomUUID()`:** as was the case for `Math.random()`, the [Web Crypto API Specification](https://w3c.github.io/webcrypto/#crypto-interface) fails to specify a concrete cryptographically secure random number generator.

| Symbol Name                  |
| ---------------------------- |
| `crypto`                     |
| `Crypto`                     |
| `Crypto.subtle`              |
| `CryptoKey`                  |
| `CryptoKey.type`             |
| `CryptoKey.extractable`      |
| `CryptoKey.algorithm`        |
| `CryptoKey.usages`           |
| `SubtleCrypto`               |
| `SubtleCrypto.encrypt()`     |
| `SubtleCrypto.decrypt()`     |
| `SubtleCrypto.sign()`        |
| `SubtleCrypto.verify()`      |
| `SubtleCrypto.digest()`      |
| `SubtleCrypto.generateKey()` |
| `SubtleCrypto.deriveKey()`   |
| `SubtleCrypto.deriveBits()`  |
| `SubtleCrypto.importKey()`   |
| `SubtleCrypto.exportKey()`   |
| `SubtleCrypto.wrapKey()`     |
| `SubtleCrypto.unwrapKey()`   |
