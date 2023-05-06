<!-- markdownlint-disable MD043 -->

# VIP-05 --- JavaScript `Encoding` Capability

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

`v-language:javascript`

`capability:Encoding`

`required-capability:Streams`

---

- [1. Capability Description](#1-capability-description)
- [2. Available Symbols](#2-available-symbols)

---

## 1. Capability Description

The `Encoding` capability makes the functionality provided by the [Encoding API](https://developer.mozilla.org/en-US/docs/Web/API/Encoding_API) available.

The `Streams` capability is **REQUIRED** for some symbols defined by this capability to be available.

## 2. Available Symbols

The following table details what symbols are made available by this capability, and whether they require the `Streams` capability or not:

| Symbol Name                                 | Requires `Streams` |
| ------------------------------------------- | ------------------ |
| `TextDecoder`                               | No                 |
| `TextDecoder.TextDecoder()`                 | No                 |
| `TextDecoder.encoding`                      | No                 |
| `TextDecoder.fatal`                         | No                 |
| `TextDecoder.ignoreBOM`                     | No                 |
| `TextDecoder.decode()`                      | No                 |
| `TextEncoder`                               | No                 |
| `TextEncoder.TextEncoder()`                 | No                 |
| `TextEncoder.prototype.encoding`            | No                 |
| `TextEncoder.encode()`                      | No                 |
| `TextEncoder.encodeInto()`                  | No                 |
| `TextDecoderStream`                         | Yes                |
| `TextDecoderStream.TextDecoderStream()`     | Yes                |
| `TextDecoderStream.encoding`                | Yes                |
| `TextDecoderStream.fatal`                   | Yes                |
| `TextDecoderStream.ignoreBOM`               | Yes                |
| `TextDecoderStream.readable`                | Yes                |
| `TextDecoderStream.writable`                | Yes                |
| `TextEncoderStream`                         | Yes                |
| `TextEncoderStream.TextEncoderStream()`     | Yes                |
| `TextEncoderStream.encoding`                | Yes                |
| `TextEncoderStream.readable`                | Yes                |
| `TextEncoderStream.writable`                | Yes                |
| `CompressionStream`                         | Yes                |
| `CompressionStream.CompressionStream()`     | Yes                |
| `CompressionStream.readable`                | Yes                |
| `CompressionStream.writable`                | Yes                |
| `DecompressionStream`                       | Yes                |
| `DecompressionStream.DecompressionStream()` | Yes                |
| `DecompressionStream.readable`              | Yes                |
| `DecompressionStream.writable`              | Yes                |
