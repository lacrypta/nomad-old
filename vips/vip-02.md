<!-- markdownlint-disable MD043 -->

# VIP-02 --- JavaScript `Async` Capability

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

`v-language:javascript`

`capability:Async`

---

- [1. Capability Description](#1-capability-description)
- [2. Available Symbols](#2-available-symbols)

---

## 1. Capability Description

The `Async` capability makes all asynchronous manipulation constructions and supporting classes available.

## 2. Available Symbols

The following table details what symbols are made available by this capability:

| Symbol Name                                       |
| ------------------------------------------------- |
| `Array.fromAsync()`                               |
| `AsyncFunction`[^async-function]                  |
| `AsyncFunction.AsyncFunction()`                   |
| `AsyncFunction.prototype.constructor`             |
| `AsyncFunction.prototype[@@toStringTag]`          |
| `AsyncGenerator`                                  |
| `AsyncGenerator.prototype.constructor`            |
| `AsyncGenerator.prototype[@@toStringTag]`         |
| `AsyncGenerator.prototype.next()`                 |
| `AsyncGenerator.prototype.return()`               |
| `AsyncGenerator.prototype.throw()`                |
| `AsyncGeneratorFunction`                          |
| `AsyncGeneratorFunction.AsyncGeneratorFunction()` |
| `AsyncGeneratorFunction.prototype.constructor`    |
| `AsyncGeneratorFunction.prototype.prototype`      |
| `AsyncGeneratorFunction.prototype[@@toStringTag]` |
| `AsyncIterator`                                   |
| `AsyncIterator.prototype[@@asyncIterator]()`      |
| `Atomics`                                         |
| `Atomics[@@toStringTag]`                          |
| `Atomics.add()`                                   |
| `Atomics.and()`                                   |
| `Atomics.compareExchange()`                       |
| `Atomics.exchange()`                              |
| `Atomics.isLockFree(size)`                        |
| `Atomics.load()`                                  |
| `Atomics.notify()`                                |
| `Atomics.or()`                                    |
| `Atomics.store()`                                 |
| `Atomics.sub()`                                   |
| `Atomics.wait()`                                  |
| `Atomics.waitAsync()`                             |
| `Atomics.xor()`                                   |
| `Promise`                                         |
| `Promise.Promise()`                               |
| `Promise[@@species]`                              |
| `Promise.all()`                                   |
| `Promise.allSettled()`                            |
| `Promise.any()`                                   |
| `Promise.race()`                                  |
| `Promise.reject()`                                |
| `Promise.resolve()`                               |
| `Promise.prototype.constructor`                   |
| `Promise.prototype[@@toStringTag]`                |
| `Promise.prototype.catch()`                       |
| `Promise.prototype.finally()`                     |
| `Promise.prototype.then()`                        |
| `SharedArrayBuffer`                               |
| `SharedArrayBuffer.SharedArrayBuffer()`           |
| `SharedArrayBuffer[@@species]`                    |
| `SharedArrayBuffer.prototype.byteLength`          |
| `SharedArrayBuffer.prototype.constructor`         |
| `SharedArrayBuffer.prototype.growable`            |
| `SharedArrayBuffer.prototype.maxByteLength`       |
| `SharedArrayBuffer.prototype[@@toStringTag]`      |
| `SharedArrayBuffer.prototype.grow()`              |
| `SharedArrayBuffer.prototype.slice()`             |
| `Symbol.asyncIterator`                            |

[^async-function]: The `AsyncFunction` global object is not predefined in JavaScript, but rather defined as `const AsyncFunction = async function () {}.constructor;` (see: [`AsyncFunction`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AsyncFunction/AsyncFunction)).
Nonetheless, clients supporting the `Async` capability **MUST** provide the `AsyncFunction` global object defined as indicated above when executing a validator with the `Async` capability.
