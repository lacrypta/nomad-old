<!-- markdownlint-disable MD043 -->

# VIP-09 --- JavaScript `Fetch` Capability

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**,  **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

`v-language:javascript`

`capability:Fetch`

`required-capability:Async`

---

- [1. Capability Description](#1-capability-description)
- [2. Available Symbols](#2-available-symbols)

---

## 1. Capability Description

The `Fetch` capability makes the functionality provided by the [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) available.

The `Async` capability is **REQUIRED** for this capability to be available.

## 2. Available Symbols

The following table details what symbols are made available by this capability:

| Symbol Name                    |
| ------------------------------ |
| `fetch()`                      |
| `Headers`                      |
| `Headers.Headers()`            |
| `Headers.append()`             |
| `Headers.delete()`             |
| `Headers.entries()`            |
| `Headers.forEach()`            |
| `Headers.get()`                |
| `Headers.getSetCookie()`       |
| `Headers.has()`                |
| `Headers.keys()`               |
| `Headers.set()`                |
| `Headers.values()`             |
| `Request`                      |
| `Request.Request()`            |
| `Request.body`                 |
| `Request.bodyUsed`             |
| `Request.cache`                |
| `Request.credentials`          |
| `Request.destination`          |
| `Request.headers`              |
| `Request.integrity`            |
| `Request.method`               |
| `Request.mode`                 |
| `Request.redirect`             |
| `Request.referrer`             |
| `Request.referrerPolicy`       |
| `Request.signal` _(Read Only)_ |
| `Request.url Read only`        |
| `Request.arrayBuffer()`        |
| `Request.blob()`               |
| `Request.clone()`              |
| `Request.formData()`           |
| `Request.json()`               |
| `Request.text()`               |
| `Response`                     |
| `Response.Response()`          |
| `Response.body`                |
| `Response.bodyUsed`            |
| `Response.headers`             |
| `Response.ok`                  |
| `Response.redirected`          |
| `Response.status`              |
| `Response.statusText`          |
| `Response.type`                |
| `Response.url`                 |
| `Response.error()`             |
| `Response.redirect()`          |
| `Response.arrayBuffer()`       |
| `Response.blob()`              |
| `Response.clone()`             |
| `Response.formData()`          |
| `Response.json()`              |
| `Response.text()`              |
