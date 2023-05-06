<!-- markdownlint-disable MD043 -->

# VIP-00 --- General VIP Guidelines

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

---

- [1. Motivation](#1-motivation)
- [2. Short Description](#2-short-description)
- [3. Overview](#3-overview)
- [4. VIP Types](#4-vip-types)
  - [4.1. Language VIPs](#41-language-vips)
  - [4.2. Capability VIPs](#42-capability-vips)
- [5. VIP List](#5-vip-list)

---

## 1. Motivation

The purpose of this VIP is to establish the general guidelines for VIP submissions.

## 2. Short Description

This VIP will set the groundwork for transparent VIP submission and review processes.

Defining clear guidelines has the inherent benefit of channeling community input in a healthy and ordered manner.

## 3. Overview

This document is organized as follows: first, we define the concept of [VIP Types](#4-vip-types), and then we set out to define the guidelines for submitting [Language](#41-language-vips) and [Capability](#42-capability-vips) VIPs, finally, we close with a [list](#5-vip-list) of current VIPs, and their defined languages and capabilities.

## 4. VIP Types

This document recognizes two _types_ of VIPs:

- **Language VIPs:** these VIPs define languages validators may use in a `"validator-language"` tag.
- **Capability VIPs:** these VIPs define capabilities validators may declare in `"validator-language"` tags and use within themselves.

### 4.1 Language VIPs

Language VIPs **MUST** define:

- The `<LANGUAGE>` placeholder value to use in a `"v-language"` tag.
- The expected semantics of the `.content` field of a `"v"`-tagged event.
- The manner in which the `event` and `tagIndex` arguments should be passed to the validator code.
- The manner in which the validator's return value should be interpreted (including any exceptional conditions that may arise).
- The _functional_ specification of the validator's execution environment.
- Any additional environmental considerations that should be observed (eg. global locale settings, seeds, etc.).
- Any pre-defined facilities a validator may use.

### 4.2 Capability VIPs

Capability VIPs **MUST** define:

- The capability identification string to use.
- The list of facilities enabled by utilizing the capability in question.
- Any dependencies incurred.

## 5. VIP List

Index of all defined VIPs:

- [VIP-01: JavaScript `"v-language"` Tag](vip-01.md)
- [VIP-02: JavaScript `Async` Capability](vip-02.md)
- [VIP-03: JavaScript `Date` Capability](vip-03.md)
- [VIP-04: JavaScript `Streams` Capability](vip-04.md)
- [VIP-05: JavaScript `Encoding` Capability](vip-05.md)
- [VIP-06: JavaScript `URL` Capability](vip-06.md)
- [VIP-07: JavaScript `Crypto` Capability](vip-07.md)
- [VIP-08: JavaScript `XMLHttpRequest` Capability](vip-08.md)
- [VIP-09: JavaScript `Fetch` Capability](vip-09.md)

List of defined `"v-language"` tags:

| Language     | VIP             |
| ------------ | --------------- |
| `javascript` | [01](vip-01.md) |

List of defined capabilities and their associated language:

| Capability       | Language     | VIP             |
| ---------------- | ------------ | --------------- |
| `Async`          | `javascript` | [02](vip-02.md) |
| `Date`           | `javascript` | [03](vip-03.md) |
| `Streams`        | `javascript` | [04](vip-04.md) |
| `Encoding`       | `javascript` | [05](vip-05.md) |
| `URL`            | `javascript` | [06](vip-06.md) |
| `Crypto`         | `javascript` | [07](vip-07.md) |
| `XMLHttpRequest` | `javascript` | [08](vip-08.md) |
| `Fetch`          | `javascript` | [09](vip-09.md) |
