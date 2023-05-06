# NIP-XX --- Immutable Badges

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

`kind:9` `kind:10`

`depends:01` `depends:16` `depends:58`
`mentions:33`

---

- [1. Motivation](#1-motivation)
- [2. Short Description](#2-short-description)
- [3. Overview](#3-overview)
- [4. Immutable Badge Definition Event](#4-immutable-badge-definition-event)
- [5. Immutable Badge Award Event](#5-immutable-badge-award-event)
  - [5.1. Fragile Badge Award Event](#51-fragile-badge-award-event)
- [6. Profile Badges Event](#6-profile-badges-event)
- [7. Client Behavior](#7-client-behavior)
  - [7.1. The Case of Fragile Badge Awards](#71-the-case-of-fragile-badge-awards)
- [8. FAQ](#8-faq)

---

## 1. Motivation

The purpose of this NIP is to provide immutable badge definitions that may not be changed once created.

## 2. Short Description

[Badge Definition Events](https://github.com/nostr-protocol/nips/blob/master/58.md#badge-definition-event) are defined as having `kind:30009`, ie. they're parameterized replaceable events (see [NIP-33](https://github.com/nostr-protocol/nips/blob/master/33.md)).
This can be seen as a feature, or a bug, depending on the use case and the ultimate purpose the end user will give to the whole badge lifecycle.

When seen as "living" awards that may change in time, a parameterized replaceable event is certainly the tool for the job.
When seen as "static" achievements, being able to change the definition of a badge post-facto is certainly not desired.

In order to address this last issue, we reserve two new kinds, and stipulate that messages of said kinds represent [Immutable Badge Definition](#4-immutable-badge-definition-event) and [Immutable Badge Award](#5-immutable-badge-award-event) events: these allow users awarded one such badge to rest assured that said badge will not change at any future point.

## 3. Overview

This document is organized as follows: first we introduce the new [Immutable Badge Definition](#4-immutable-badge-definition-event) and [Immutable Badge Award](#5-immutable-badge-award-event) events, next we deal with changes to the [Profile Badges Event](#6-profile-badges-event), finally we deal with [Client Behavior](#7-client-behavior).
We close with a short [FAQ](#8-faq) section.

## 4. Immutable Badge Definition Event

An _immutable badge definition_ event is defined as en event of `kind:9` of the following form:

```json
{
    ...,
    "kind": 9,
    ...,
    "tags": [
        ...,
        ["name", <BADGE_NAME>],
        ...,
        ["description", <DESCRIPTION_TEXT>]
        ...,
        ["image", <IMAGE_URL>, <WIDTH_X_HEIGHT>],
        ...,
        ["thumb", <THUMBNAIL_URL_1>, <WIDTH_X_HEIGHT_1>],
        ...,
        ["thumb", <THUMBNAIL_URL_2>, <WIDTH_X_HEIGHT_2>],
        ...
    ],
    ...
}
```

Where the following tags **MAY** be present:

- **`"name"`:** a short name for the badge.
- **`"description"`:** a textual representation of the image, the meaning behind the badge, or the reason of it's issuance.
- **`"image"`:** the URL of a high-resolution image representing the badge.
  The second value optionally specifies the dimensions of the image as `<WIDTH>x<HEIGHT>` in pixels.
  Badge recommended dimensions is `1024x1024` pixels.
- **`"thumb"`:** an URL pointing to a thumbnail version of the image referenced in the `"image"` tag.
  The second value optionally specifies the dimensions of the thumbnail as `<WIDTH>x<HEIGHT>` in pixels.
  Zero or more `"thumb"` tags **MAY** be given.

(these definition were lifted verbatim from the [NIP-58](https://github.com/nostr-protocol/nips/blob/master/58.md) ones, the [recommendations](https://github.com/nostr-protocol/nips/blob/master/58.md#recommendations) therein are applicable here as well).

The event's `.content` field **MAY** contain whatever content the issuer desires, and **SHOULD** be ignored for the purposes of this NIP.

> Note that according to [NIP-16](https://github.com/nostr-protocol/nips/blob/master/16.md) immutable badge definition events should be stored and **MUST NOT** be replaced at all.

## 5. Immutable Badge Award Event

An _immutable badge award_ event is defined as en event of `kind:10` of the following form:

```json
{
    ...
    "kind": 10,
    ...,
    "tags": [
        ...,
        ["e", <IMMUTABLE_BADGE_DEFINITION_EVENT_ID>],  // kind:9
        ...,
        ["p", <RECIPIENT_PUBKEY_1>],
        ...,
        ["p", <RECIPIENT_PUBKEY_2>],
        ...
    ],
    ...
}
```

Where the following tags **MUST** be present:

- **`"e"`:** this tag references the `kind:9` immutable badge definition event being awarded.
  Clients **SHOULD** validate that the event referred to has the same `.pubkey` value as this event.
- **`"p"`:** one or more of these tags reference the public keys of the users being awarded the tag referred to above.

The event's `.content` field **MAY** contain whatever content the issuer/awarder desires, and **SHOULD** be ignored for the purposes of this NIP.

> Note that according to [NIP-16](https://github.com/nostr-protocol/nips/blob/master/16.md) immutable badge award events should be stored and **MUST NOT** be replaced at all.

An additional award event is defined for special cases treatment.

Note that the main difference from [NIP-58's Badge Award Event](https://github.com/nostr-protocol/nips/blob/master/58.md#badge-award-event) is that our definition uses `"e"` tags to identify the (Immutable) Badge Definition Event, whereas [NIP-58](https://github.com/nostr-protocol/nips/blob/master/58.md)'s one uses an `"a"` tag for this purpose.

### 5.1. Fragile Badge Award Event

A _fragile_ badge award event is defined as an event of `kind:10` of the following form:

```json
{
    ...
    "kind": 10,
    ...,
    "tags": [
        ...,
        ["e", <BADGE_DEFINITION_EVENT_ID>],  // kind:30009
        ...,
        ["p", <RECIPIENT_PUBKEY_1>],
        ...,
        ["p", <RECIPIENT_PUBKEY_2>],
        ...
    ],
    ...
}
```

Where the _only_ differences from an immutable badge award event are that the event referred to by the mandatory `"e"` tag can be of `kind:30009` instead of `kind:9`.
This allows for an immutable badge award event to refer to a [NIP-58](https://github.com/nostr-protocol/nips/blob/master/58.md)-style badge.

> Note that according to [NIP-16](https://github.com/nostr-protocol/nips/blob/master/16.md) fragile badge award events should be stored and **MUST NOT** be replaced at all.

## 6. Profile Badges Event

Profile badges events as defined in [NIP-58](https://github.com/nostr-protocol/nips/blob/master/58.md) are extended herein.

These are events of `kind:30008`, having a `"d"` tag value equal to `"badges"` (cf. `"profile_badges"` in [NIP-58](https://github.com/nostr-protocol/nips/blob/master/58.md)), of the following form:

```json
{
    ...,
    "kind": 30008,
    ...,
    "tags": [
        ...,
        ["d", "badges"],
        ...,
        ["e", <IMMUTABLE_BADGE_DEFINITION_EVENT_ID>],  // kind:9
        ["e", <IMMUTABLE_BADGE_AWARD_EVENT_ID>],  // kind:10
        ...,
        ["a", <BADGE_DEFINITION_LINK>],  // 30009:<ISSUER_PUBKEY>:<BADGE_ID>
        ["e", <BADGE_AWARDS_EVENT_ID>],  // kind:8
        ...
    ],
    ...
}
```

Where the following tags **MUST** be present:

- **`"d"`:** the value **MUST** equal the unique identifier `"badges"`.

Additionally, there may be zero or more ordered consecutive pairs of either of these tags:

- **`"e"` / `"e"`:** in this configuration, the first `"e"` tag refers to an immutable badge definition event (ie. `kind:9`), the second `"e"` tag refers to an immutable (or fragile) badge award event (ie. `kind:10`).
  In turn, the `"e"` tag of the event referred to by this pair's second `"e"` tag **MUST** equal this pair's first `"e"` tag.
- **`"a"` / `"e"`:** in this configuration, the `"a"` tag refers to a [NIP-58](https://github.com/nostr-protocol/nips/blob/master/58.md) badge definition event (ie. the value of the `"a"` tag should be of the form `"30009:<ISSUER_PUBKEY>:<BADGE_ID>"`), and the `"e"` tag refers to a badge award event (ie. `kind:8`).
  In turn, the `"a"` tag of the event referred to by this pair's `"e"` tag **MUST** equal this pair's `"a"` tag.

## 7. Client Behavior

Clients are encouraged to have a `kind:30008` event with a `"d"` tag of `"badges"` take precedence over a `kind:30008` event with a `"d"` tag of `"profile_badges"`, so that the richer information provided in this new event be rendered if available, and only revert to the legacy one if no `"badges"` alternative is found.

Rendering and ordering client recommendations are unchanged from those outlined in [NIP-58](https://github.com/nostr-protocol/nips/blob/master/58.md).

The only changes needed are the parsing of **`"e"` / `"e"`** tag pairs in addition to **`"a"` / `"e"`** ones.

### 7.1. The Case of Fragile Badge Awards

[Fragile Badge Awards](#51-fragile-badge-award-event) are designed to afford the profile user a measure of control over what to do when a badge they're displaying changes.

The rationale is that a fragile award should act as a promise of sorts, from the issuer, that a _specific version_ of an updatable badge is to be awarded to the user; upon the definition being updated, the event ID being referred to should no longer exist, and said award should no longer be valid.

Thus, fragile badge awards provide assurances that the awarded badge will either not change, or if it does, will no longer be applicable to the user.

Having said all that, it has been our experience that relays don't usually discard parameterized replaceable events, which makes the event ID originally awarded persist in time for longer than strictly mandated by [NIP-33](https://github.com/nostr-protocol/nips/blob/master/33.md).

## 8. FAQ

**Why prevent badges from being modified? Can we not simply not modify them and leave it at that?**

Users are still free to define badges with a `kind:30009` event and simply not replace said event at all, this is not what this NIP addresses.
Rather, it _prevents_ any modification whatsoever to the badge definition itself.

There's a difference between being able to change something and no doing it, and having a guarantee that it will not ever change.
This NIP addresses the latter.
