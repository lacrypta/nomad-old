<!-- markdownlint-disable MD043 -->

# VIP-08 --- JavaScript `XMLHttpRequest` Capability

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**,  **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

`v-language:javascript`

`capability:XMLHttpRequest`

`required-capability:Async`

---

> &#x26a0; **NOTE:** this should be considered a _LAST RESORT_ capability to use, the ability to perform arbitrary calls to the outside world is both unwieldy and dangerous.
> It is recommended to segregate access to specific external services into specific capabilities, and use those narrower interfaces in favor of this blunter instrument.

---

- [1. Capability Description](#1-capability-description)
- [2. Available Symbols](#2-available-symbols)

---

## 1. Capability Description

The `XMLHttpRequest` capability makes the functionality provided by the [XMLHttpRequest API](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) available.

The `Async` capability is **REQUIRED** for this capability to be available.

## 2. Available Symbols

The following table details what symbols are made available by this capability:

| Symbol Name                              |
| ---------------------------------------- |
| `FormData`                               |
| `FormData.FormData()`                    |
| `FormData.append()`                      |
| `FormData.delete()`                      |
| `FormData.entries()`                     |
| `FormData.get()`                         |
| `FormData.getAll()`                      |
| `FormData.has()`                         |
| `FormData.keys()`                        |
| `FormData.set()`                         |
| `FormData.values()`                      |
| `ProgressEvent`                          |
| `ProgressEvent.ProgressEvent()`          |
| `ProgressEvent.lengthComputable`         |
| `ProgressEvent.loaded`                   |
| `ProgressEvent.total`                    |
| `XMLHttpRequest`                         |
| `XMLHttpRequest.XMLHttpRequest()`        |
| `XMLHttpRequest.readyState`              |
| `XMLHttpRequest.response`                |
| `XMLHttpRequest.responseText`            |
| `XMLHttpRequest.responseType`            |
| `XMLHttpRequest.responseURL`             |
| `XMLHttpRequest.responseXML`             |
| `XMLHttpRequest.status`                  |
| `XMLHttpRequest.statusText`              |
| `XMLHttpRequest.timeout`                 |
| `XMLHttpRequest.upload`                  |
| `XMLHttpRequest.withCredentials`         |
| `XMLHttpRequest.abort()`                 |
| `XMLHttpRequest.getAllResponseHeaders()` |
| `XMLHttpRequest.getResponseHeader()`     |
| `XMLHttpRequest.open()`                  |
| `XMLHttpRequest.overrideMimeType()`      |
| `XMLHttpRequest.send()`                  |
| `XMLHttpRequest.setRequestHeader()`      |
