# NIP-XX --- Immutable Badges

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**,  **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

`depends:01` `depends:58`
`mentions:33`

---

- [1. Motivation](#1-motivation)
- [2. Short Description](#2-short-description)
- [3. Overview](#3-overview)
- [4. Badge Award Event Content Semantic](#4-badge-award-event-content-semantic)
- [5. Client Behavior](#5-client-behavior)
- [6. FAQ](#6-faq)

---

## 1. Motivation

The purpose of this NIP is to provide immutable badge definitions that may not be changed once created.

## 2. Short Description

[Badge Definition Events](https://github.com/nostr-protocol/nips/blob/master/58.md#badge-definition-event) are defined as having `kind:30009`, ie. they're parameterized replaceable events (see [NIP-33](https://github.com/nostr-protocol/nips/blob/master/33.md)).
This can be seen as a feature, or a bug, depending on the use case and the ultimate purpose the end user will give to the whole badge lifecycle.

When seen as "living" awards that may change in time, a parameterized replaceable event is certainly the tool for the job.
When seen as "static" achievements, being able to change the definition of a badge post-facto is certainly not desired.

Noting that [NIP-58](https://github.com/nostr-protocol/nips/blob/master/58.md#badge-award-event) fails to assign a semantic meaning to the [Badge Award Event](https://github.com/nostr-protocol/nips/blob/master/58.md#badge-award-event)'s `.content` field, we shall take advantage of that and use it to "freeze" the badge definition being awarded.

## 3. Overview

This document is organized as follows: we introduce the new [Badge Award Event Content Semantic](#4-badge-award-event-content-semantic), finally we deal with [Client Behavior](#5-client-behavior).
We close with a short [FAQ](#6-faq) section.

## 4. Badge Award Event Content Semantic

[NIP-58](https://github.com/nostr-protocol/nips/blob/master/58.md) gives no semantic to a Badge Award Event's `.content` field.
We make use of this oversight by stipulating that a badge award event may come in one of two forms: it may either be _"empty"_ or _"binding"_.

Empty badge award events have the form:

```json
{
    ...
    "kind": 10,
    ...,
    "tags": [
        ...,
        ["a", <BADGE_DEFINITION_EVENT_ID>],  // kind:30009
        ...,
        ["p", <RECIPIENT_PUBKEY_1>],
        ...,
        ["p", <RECIPIENT_PUBKEY_2>],
        ...
    ],
    ...,
    "content": "",
    ...
}
```

(note that this is exactly the same as the ones proposed in [NIP-58](https://github.com/nostr-protocol/nips/blob/master/58.md)); on the other hand, binding badge award events have the form:

```json
{
    ...
    "kind": 10,
    ...,
    "tags": [
        ...,
        ["a", <BADGE_DEFINITION_EVENT_ID>],  // kind:30009
        ...,
        ["p", <RECIPIENT_PUBKEY_1>],
        ...,
        ["p", <RECIPIENT_PUBKEY_2>],
        ...
    ],
    ...,
    "content": <BADGE_DEFINITION_AWARD_EVENT>,
    ...
}
```

where the `<BADGE_DEFINITION_AWARD_EVENT>` placeholder stands for the JSON string of the version of the event referred to by the `"a"` tag being in effect awarded at this time.

The `.kind`, `.pubkey`, and `"d"` tag of the serialized event in the `.content` field **MUST** be compatible with the value of the `"a"` tag in the event's tags.

## 5. Client Behavior

Conforming clients need to check the `.content` field of a badge award event mentioned in the `"e"` tags of a `kind:30008` event with a `"d"` tag with `"profile_badges"` value.
If a non-empty `.content` field is found, conforming clients **MUST** render the event encoded therein instead of looking for the referred element by `"a"` tag.

## 6. FAQ

**Why prevent badges from being modified? Can we not simply not modify them and leave it at that?**

Users are still free to define badges with a `kind:30009` event and simply not replace said event at all, this is not what this NIP addresses.
Rather, it _prevents_ any modification whatsoever to the badge definition itself.

There's a difference between being able to change something and no doing it, and having a guarantee that it will not ever change.
This NIP addresses the latter.
