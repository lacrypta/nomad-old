# NIP-XX --- NIP-05 External Identity Claim

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**,  **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

`depends:05` `depends:39`

---

- [1. Motivation](#1-motivation)
- [2. Short Description](#2-short-description)
- [3. Overview](#3-overview)
- [4. NIP-05 Claim Type](#4-nip-05-claim-type)
- [5. Client Behavior](#5-client-behavior)

---

## 1. Motivation

The purpose of this NIP is to provide means for users to establish their identity against one or more [NIP-05](https://github.com/nostr-protocol/nips/blob/master/05.md) aliases.

## 2. Short Description

The idea behind this NIP is to allow users to establish equivalences between their multiple identities using the same underlying public key.

In order to do this, we allow a new claim type to be used in an `"i"` tag.

## 3. Overview

This document is organized as follows: first we introduce the new [NIP-05 Claim Type](#4-nip-05-claim-type), finally we deal with the [Client Behavior](#5-client-behavior).

## 4. NIP-05 Claim Type

A _NIP-05 claim type_ is a new claim type (as defined in [NIP-39](https://github.com/nostr-protocol/nips/blob/master/39.md)) where:

- **platform:** is the `nip05` string
- **identity:** is a `<local-part>` as defined in [NIP-05](https://github.com/nostr-protocol/nips/blob/master/05.md)
- **proof:** is a `<domain>` as defined in [NIP-05](https://github.com/nostr-protocol/nips/blob/master/05.md)

Example:

```json
[
    "i",
    "nip05:<USERNAME>",
    "<DOMAIN>"
]
```

## 5. Client Behavior

Clients **MAY** validate this identity tag by querying `https://<DOMAIN>/.well-known/nostr.json?name=<USERNAME>`, and verifying that in the resulting message:

```json
{
    "names": {
        "<USERNAME>": <PUBKEY>
    }
}
```

the `<PUBKEY>` corresponds to the event's `.pubkey` field.
