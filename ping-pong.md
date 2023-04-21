<!-- markdownlint-enable -->
<!-- markdownlint-disable MD013 -->

# NIP-XX

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**,  **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

`kind:20202`

`depends:01` `depends:16`

---

- [1. Motivation](#1-motivation)
- [2. Short Description](#2-short-description)
- [3. Overview](#3-overview)
- [4. Ping / Pong Events](#4-ping--pong-events)
- [5. Client Behavior](#5-client-behavior)
- [6. FAQ](#6-faq)

---

## 1. Motivation

Te purpose of this NIP is to provide real-time, user-initiated, alerting from one user to one or more others, and back.

## 2. Short Description

The basic idea behind this NIP is to implement a form of "ping" (or "nudge", or "poke") from one user to one or more others, and a form for the target users to acknowledge the ping.

To that end, a new ephemeral kind is reserved to realize this new message types, and the internal structure of it modeled around already existing tags.

## 3. Overview

This document is organized as follows: first we introduce the new [Ping / Pong Events](#4-ping--pong-events), finally we deal with the [Client Behavior](#5-client-behavior).
We close with a short [FAQ](#6-faq) section.

## 4. Ping / Pong Events

A _ping event_ is defined as an event with `kind:20202`.

A ping event's `.content` field **SHOULD** be ignored for the purposes of this NIP, but future iterations may provide semantics for it.

A ping event's `"p"` tags list all the users to be notified, or "pinged".

An event of `kind:20202` having one or more `"e"` tags referring to other `kind:20202` events is termed a _pong event_.
The semantics of a pong event are that of an acknowledgement to the mentioned ping events.

Thus, a user may acknowledge several pings at once.

> Note that according to [NIP-16](https://github.com/nostr-protocol/nips/blob/master/16.md) ping / pong events should be sent to all matching filters and **MUST NOT** be stored at all.

## 5. Client Behavior

Upon receiving a ping event, clients are expected to notify the user, allowing for them to acknowledge the ping with a pong.

In an effort to minimize spam, clients are encouraged to only pay attention to pings from users in the user's contact list, but are not required to do so.

Clients are free to impose any manner of additional restrictions (ie. like rate limiting, or Proof-of-Work, or single target pings, etc...)

## 6. FAQ

**Why would even consider unleashing such annoying a bane on the user's attention?**

Because we believe in the user's autonomy to configure and determine when pings should be noticed and / or acknowledged.
As clients extend support for this NIP, we expect them to implement the necessary safeguards to prevent spam generation and propagation.
Likewise, relays are expected to moderate the ping traffic on their side as well.
