# NIP-XX

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**,  **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

`kind:9`

`depends:01` `depends:16` `depends:58`
`mentions:33`

---

- [1. Motivation](#1-motivation)
- [2. Short Description](#2-short-description)
- [3. Overview](#3-overview)
- [4. Immutable Badge Definition Event](#4-immutable-badge-definition-event)
- [5. Client Behavior](#5-client-behavior)
- [6. FAQ](#6-faq)

---

## 1. Motivation

Te purpose of this NIP is to provide immutable badge definitions that may not be changed once created.

## 2. Short Description

[Badge Definition Events](https://github.com/nostr-protocol/nips/blob/master/58.md#badge-definition-event) are defined as having `kind:30009`, ie. they're parameterized replaceable events (see [NIP-33](https://github.com/nostr-protocol/nips/blob/master/33.md)).
This can be seen as a feature, or a bug, depending on the use case and the ultimate purpose the end user will give to the whole badge lifecycle.

When seen as "living" awards that may change in time, a parameterized replaceable event is certainly the tool for the job.
When seen as "static" achievements, being able to change the definition of a badge post-facto is certainly not desired.

In order to address this last issue, we reserve a new kind, and stipulate that messages of said kind represent [Immutable Badge Definition Events](#4-immutable-badge-definition-event): this allows users awarded one such badge to rest assured that said badge will not change at any future point.

## 3. Overview

This document is organized as follows: first we introduce the new [Immutable Badge Definition Event](#4-immutable-badge-definition-event), finally we deal with the [Client Behavior](#5-client-behavior).
We close with a short [FAQ](#6-faq) section.

## 4. Immutable Badge Definition Event

An _immutable badge definition event_ is defined as an event with `kind:9`.

For ease of interoperability with existing client implementations, immutable badge definition events follow ALL guidelines [Badge Definition Events](https://github.com/nostr-protocol/nips/blob/master/58.md#badge-definition-event) must follow, save for the usage of a different kind.
This includes the usage of the `"d"` tag.

> Note that according to [NIP-16](https://github.com/nostr-protocol/nips/blob/master/16.md) immutable badge definition events should be stored and **MUST NOT** be replaced at all.

## 5. Client Behavior

Other than the behavioral changes induced by the usage of a different kind, clients **MUST** follow the guidelines set out in [NIP-58](https://github.com/nostr-protocol/nips/blob/master/58.md).
Nothing, other than querying for a new kind, should change client-side.

There _is_ however the issue of collisions: the existence of both a `kind:30009` and `kind:9` event for the same author and `"d"` tag.
In such a case, `kind:9` event always take precedence.
This is so that the promise `kind:9` makes can be upheld, and still have a form of interoperability with `kind:30009`.

## 6. FAQ

**Why prevent badges from being modified? Can we not simply not modify them and leave it at that?**

Users are still free to define badges with a `kind:30009` event and simply not replace said event at all, this is not what this NIP addresses.
Rather, it _prevents_ any modification whatsoever to the badge definition itself.

There's a difference between being able to change something and no doing it, and having a guarantee that it will not ever change.
This NIP addresses the latter.

**Why do `kind:9` events take precedence over `kind:30009` in case of collisions?**

Because `kind:9` PROMISES that once its existence is known, no changes can be made to the badge definition, allowing for `kind:30009` to override that would make liars of ourselves.

On the other hand, establishing that particular precedence allows for badges to change for a time (by using several `kind:30009` events), and then suddenly settle on a definite version (broadcasting a `kind:9` event).
