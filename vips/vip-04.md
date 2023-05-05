<!-- markdownlint-disable MD043 -->

# VIP-04 --- JavaScript `Streams` Capability

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**,  **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

`v-language:javascript`

`capability:Streams`

`required-capability:Async`

---

- [1. Capability Description](#1-capability-description)
- [2. Available Symbols](#2-available-symbols)

---

## 1. Capability Description

The `Streams` capability makes the functionality provided by the [Streams API](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) available.

The `Async` capability is **REQUIRED** for this capability to be available.

## 2. Available Symbols

The following table details what symbols are made available by this capability:

| Symbol Name                                                 |
| ----------------------------------------------------------- |
| `ReadableStream`                                            |
| `ReadableStream.ReadableStream()`                           |
| `ReadableStream.locked`                                     |
| `ReadableStream.cancel()`                                   |
| `ReadableStream.getReader()`                                |
| `ReadableStream.pipeThrough()`                              |
| `ReadableStream.pipeTo()`                                   |
| `ReadableStream.tee()`                                      |
| `ReadableStreamDefaultReader`                               |
| `ReadableStreamDefaultReader.ReadableStreamDefaultReader()` |
| `ReadableStreamDefaultReader.closed`                        |
| `ReadableStreamDefaultReader.cancel()`                      |
| `ReadableStreamDefaultReader.read()`                        |
| `ReadableStreamDefaultReader.releaseLock()`                 |
| `ReadableStreamDefaultController`                           |
| `ReadableStreamDefaultController.desiredSize`               |
| `ReadableStreamDefaultController.close()`                   |
| `ReadableStreamDefaultController.enqueue()`                 |
| `ReadableStreamDefaultController.error()`                   |
| `WritableStream`                                            |
| `WritableStream.WritableStream()`                           |
| `WritableStream.locked`                                     |
| `WritableStream.abort()`                                    |
| `WritableStream.close()`                                    |
| `WritableStream.getWriter()`                                |
| `WritableStreamDefaultWriter`                               |
| `WritableStreamDefaultWriter.WritableStreamDefaultWriter()` |
| `WritableStreamDefaultWriter.closed`                        |
| `WritableStreamDefaultWriter.desiredSize`                   |
| `WritableStreamDefaultWriter.ready`                         |
| `WritableStreamDefaultWriter.abort()`                       |
| `WritableStreamDefaultWriter.close()`                       |
| `WritableStreamDefaultWriter.releaseLock()`                 |
| `WritableStreamDefaultWriter.write()`                       |
| `WritableStreamDefaultController`                           |
| `WritableStreamDefaultController.signal`                    |
| `WritableStreamDefaultController.error()`                   |
| `TransformStream`                                           |
| `TransformStream.TransformStream()`                         |
| `TransformStream.readable`                                  |
| `TransformStream.writable`                                  |
| `TransformStreamDefaultController`                          |
| `TransformStreamDefaultController.desiredSize`              |
| `TransformStreamDefaultController.enqueue()`                |
| `TransformStreamDefaultController.error()`                  |
| `TransformStreamDefaultController.terminate()`              |
| `ByteLengthQueuingStrategy`                                 |
| `ByteLengthQueuingStrategy.ByteLengthQueuingStrategy()`     |
| `ByteLengthQueuingStrategy.highWaterMark`                   |
| `ByteLengthQueuingStrategy.size()`                          |
| `CountQueuingStrategy`                                      |
| `CountQueuingStrategy.CountQueuingStrategy()`               |
| `CountQueuingStrategy.highWaterMark`                        |
| `CountQueuingStrategy.size()`                               |
| `ReadableStreamBYOBReader`                                  |
| `ReadableStreamBYOBReader.ReadableStreamBYOBReader()`       |
| `ReadableStreamBYOBReader.closed`                           |
| `ReadableStreamBYOBReader.cancel()`                         |
| `ReadableStreamBYOBReader.read()`                           |
| `ReadableStreamBYOBReader.releaseLock()`                    |
| `ReadableByteStreamController`                              |
| `ReadableByteStreamController.byobRequest`                  |
| `ReadableByteStreamController.desiredSize`                  |
| `ReadableByteStreamController.close()`                      |
| `ReadableByteStreamController.enqueue()`                    |
| `ReadableByteStreamController.error()`                      |
| `ReadableStreamBYOBRequest`                                 |
| `ReadableStreamBYOBRequest.view`                            |
| `ReadableStreamBYOBRequest.respond()`                       |
| `ReadableStreamBYOBRequest.respondWithNewView()`            |
