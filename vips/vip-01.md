<!-- markdownlint-disable MD043 -->

# VIP-01 --- JavaScript `"v-language"` Tag

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**,  **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

`v-language:javascript`

`capability:Async`
`capability:Date`
`capability:Streams`
`capability:Encoding`
`capability:CompressionStreams`
`capability:URL`
`capability:URLPattern`
`capability:Crypto`
`capability:XMLHttpRequest`
`capability:Fetch`

---

- [1. General Conventions](#1-general-conventions)
- [2. `NostrRead` Capability](#2-nostrread-capability)
- [3. `NostrValidate` Capability](#3-nostrvalidate-capability)
- [4. Additional Capabilities](#4-additional-capabilities)

---

## 1. General Conventions

The `<LANGUAGE>` placeholder **MUST** be `"javascript"`.

The validator definition event's `.content` **MUST** be ES6-compliant.

Return values are interpreted as JavaScript booleans.
Uncaught exceptions are considered `false` return values.

The execution environment will vary depending on the actual type of execution machine (ie. NodeJS vs browser-based), but the execution tactic should be compatible with extracting the value of the following expression:

```javascript
try {
  return Function(
    '"use strict";' + validatorEvent.content,
  ).apply(
    {},
    JSON.stringify([event, tagIndex]),
  );
} catch {
  return false;
}
```

where `event` and `tagIndex` are as above, and `validatorEvent` is the validator definition event referred to by ID in the event's `"v"` tag.

> If the client supports the `Async` capability (see below) and the validator requests the `Async` capability, this expression becomes:
>
> ```javascript
> try {
>   return await AsyncFunction(
>     '"use strict";' + validatorEvent.content,
>   ).apply(
>     {},
>     JSON.stringify([event, tagIndex]),
>   );
> } catch {
>   return false;
> }
> ```

The [`DefaultLocale()`](https://402.ecma-international.org/4.0/#sec-defaultlocale) abstract operation **MUST** return the `C` locale.

No support is explicitly provided for JavaScript Events, but conforming clients are free to define a capability (perhaps called `Events`) to signal usage and support for this functionality.

## 2. `NostrRead` Capability

The **`NostrRead`** capability is implemented via a global class `NOSTR`, containing methods:

- **`NOSTR.read(filters)`:** where `filters` is an array of objects realizing the filter specification required.
- **`NOSTR.read(filters, relayUrl)`:** where `filters` is an array of objects realizing the filter specification required, and `relayUrl` is a string containing the relay URL to use.

## 3. `NostrValidate` Capability

The **`NostrValidate`** capability is implemented via a global class `NOSTR`, containing method:

- **`NOSTR.validate(event)`:** where `event` is an object containing a NOSTR event.

## 4. Additional Capabilities

The available `<CAPABILITY>` values are:

- **`Async`:** makes all asynchronous manipulation constructions and supporting classes available.
- **`Date`:** makes the `Date.Date()` constructor and the `Date.now()` static method available.
- **`Streams`:** makes the functionality provided by the [Streams API](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) available
- **`Encoding`:** makes the functionality provided by the [Encoding API](https://developer.mozilla.org/en-US/docs/Web/API/Encoding_API) available
- **`CompressionStreams`:** makes the functionality provided by the [Compression Streams API](https://developer.mozilla.org/en-US/docs/Web/API/Compression_Streams_API) available (requires the `Streams` capability to be enabled as well)
- **`URL`:** makes the functionality provided by the [URL API](https://developer.mozilla.org/en-US/docs/Web/API/URL_API) available
- **`URLPattern`:** makes the functionality provided by the [URL Pattern API](https://developer.mozilla.org/en-US/docs/Web/API/URL_Pattern_API) available
- **`Crypto`:** makes the functionality provided by the [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API) available (requires the `Async` capability to be enabled as well)
- **`XMLHttpRequest`:** makes the functionality provided by the [XMLHttpRequest API](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) available
- **`Fetch`:** makes the functionality provided by the [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) available (requires the `Async` capability to be enabled as well)

This list is not exhaustive, and clients **MAY** provide additional JavaScript capabilities if so desired.

The following table details what symbols are made available by each capability, or made available by default (note the absence of `globalThis` and `eval()`, these would enable the validator code to potentially break free from containment and are thus simply not provided at all):

| Symbol Name                                                 | Capability       | Dependencies |
| ----------------------------------------------------------- | ---------------- | ------------ |
| `Infinity`                                                  | -                | -            |
| `NaN`                                                       | -                | -            |
| `undefined`                                                 | -                | -            |
| `isFinite()`                                                | -                | -            |
| `isNaN()`                                                   | -                | -            |
| `parseFloat()`                                              | -                | -            |
| `parseInt()`                                                | -                | -            |
| `decodeURI()`                                               | -                | -            |
| `decodeURIComponent()`                                      | -                | -            |
| `encodeURI()`                                               | -                | -            |
| `encodeURIComponent()`                                      | -                | -            |
| `Object`                                                    | -                | -            |
| `Object.Object()`                                           | -                | -            |
| `Object.assign()`                                           | -                | -            |
| `Object.create()`                                           | -                | -            |
| `Object.defineProperties()`                                 | -                | -            |
| `Object.defineProperty()`                                   | -                | -            |
| `Object.entries()`                                          | -                | -            |
| `Object.freeze()`                                           | -                | -            |
| `Object.fromEntries()`                                      | -                | -            |
| `Object.getOwnPropertyDescriptor()`                         | -                | -            |
| `Object.getOwnPropertyDescriptors()`                        | -                | -            |
| `Object.getOwnPropertyNames()`                              | -                | -            |
| `Object.getOwnPropertySymbols()`                            | -                | -            |
| `Object.getPrototypeOf()`                                   | -                | -            |
| `Object.hasOwn()`                                           | -                | -            |
| `Object.is()`                                               | -                | -            |
| `Object.isExtensible()`                                     | -                | -            |
| `Object.isFrozen()`                                         | -                | -            |
| `Object.isSealed()`                                         | -                | -            |
| `Object.keys()`                                             | -                | -            |
| `Object.preventExtensions()`                                | -                | -            |
| `Object.seal()`                                             | -                | -            |
| `Object.setPrototypeOf()`                                   | -                | -            |
| `Object.values()`                                           | -                | -            |
| `Object.prototype.constructor`                              | -                | -            |
| `Object.prototype.hasOwnProperty()`                         | -                | -            |
| `Object.prototype.isPrototypeOf()`                          | -                | -            |
| `Object.prototype.propertyIsEnumerable()`                   | -                | -            |
| `Object.prototype.toLocaleString()`                         | -                | -            |
| `Object.prototype.toString()`                               | -                | -            |
| `Object.prototype.valueOf()`                                | -                | -            |
| `Function`                                                  | -                | -            |
| `Function.Function()`                                       | -                | -            |
| `Function.prototype.constructor`                            | -                | -            |
| `Function.length`                                           | -                | -            |
| `Function.name`                                             | -                | -            |
| `Function.prototype`                                        | -                | -            |
| `Function.prototype.apply()`                                | -                | -            |
| `Function.prototype.bind()`                                 | -                | -            |
| `Function.prototype.call()`                                 | -                | -            |
| `Function.prototype.toString()`                             | -                | -            |
| `Boolean`                                                   | -                | -            |
| `Boolean.Boolean()`                                         | -                | -            |
| `Boolean.prototype.constructor`                             | -                | -            |
| `Boolean.prototype.toString()`                              | -                | -            |
| `Boolean.prototype.valueOf()`                               | -                | -            |
| `Symbol`                                                    | -                | -            |
| `Symbol.Symbol()`                                           | -                | -            |
| `Symbol.hasInstance`                                        | -                | -            |
| `Symbol.isConcatSpreadable`                                 | -                | -            |
| `Symbol.iterator`                                           | -                | -            |
| `Symbol.match`                                              | -                | -            |
| `Symbol.matchAll`                                           | -                | -            |
| `Symbol.replace`                                            | -                | -            |
| `Symbol.search`                                             | -                | -            |
| `Symbol.species`                                            | -                | -            |
| `Symbol.split`                                              | -                | -            |
| `Symbol.toPrimitive`                                        | -                | -            |
| `Symbol.toStringTag`                                        | -                | -            |
| `Symbol.unscopables`                                        | -                | -            |
| `Symbol.for()`                                              | -                | -            |
| `Symbol.keyFor()`                                           | -                | -            |
| `Symbol.prototype.constructor`                              | -                | -            |
| `Symbol.prototype.description`                              | -                | -            |
| `Symbol.prototype[@@toStringTag]`                           | -                | -            |
| `Symbol.prototype.toString()`                               | -                | -            |
| `Symbol.prototype.valueOf()`                                | -                | -            |
| `Symbol.prototype[@@toPrimitive]()`                         | -                | -            |
| `Error`                                                     | -                | -            |
| `Error.Error()`                                             | -                | -            |
| `Error.prototype.constructor`                               | -                | -            |
| `Error.prototype.name`                                      | -                | -            |
| `Error.cause`                                               | -                | -            |
| `Error.message`                                             | -                | -            |
| `Error.prototype.toString()`                                | -                | -            |
| `AggregateError`                                            | -                | -            |
| `AggregateError.AggregateError()`                           | -                | -            |
| `AggregateError.prototype.constructor`                      | -                | -            |
| `AggregateError.prototype.name`                             | -                | -            |
| `AggregateError.errors`                                     | -                | -            |
| `RangeError`                                                | -                | -            |
| `RangeError.RangeError()`                                   | -                | -            |
| `RangeError.prototype.constructor`                          | -                | -            |
| `RangeError.prototype.name`                                 | -                | -            |
| `ReferenceError`                                            | -                | -            |
| `ReferenceError.ReferenceError()`                           | -                | -            |
| `ReferenceError.prototype.constructor`                      | -                | -            |
| `ReferenceError.prototype.name`                             | -                | -            |
| `TypeError`                                                 | -                | -            |
| `TypeError.TypeError()`                                     | -                | -            |
| `TypeError.prototype.constructor`                           | -                | -            |
| `TypeError.prototype.name`                                  | -                | -            |
| `URIError`                                                  | -                | -            |
| `URIError.URIError()`                                       | -                | -            |
| `URIError.prototype.constructor`                            | -                | -            |
| `URIError.prototype.name`                                   | -                | -            |
| `Number`                                                    | -                | -            |
| `Number.Number()`                                           | -                | -            |
| `Number.EPSILON`                                            | -                | -            |
| `Number.MAX_SAFE_INTEGER`                                   | -                | -            |
| `Number.MAX_VALUE`                                          | -                | -            |
| `Number.MIN_SAFE_INTEGER`                                   | -                | -            |
| `Number.MIN_VALUE`                                          | -                | -            |
| `Number.NaN`                                                | -                | -            |
| `Number.NEGATIVE_INFINITY`                                  | -                | -            |
| `Number.POSITIVE_INFINITY`                                  | -                | -            |
| `Number.isFinite()`                                         | -                | -            |
| `Number.isInteger()`                                        | -                | -            |
| `Number.isNaN()`                                            | -                | -            |
| `Number.isSafeInteger()`                                    | -                | -            |
| `Number.parseFloat()`                                       | -                | -            |
| `Number.parseInt()`                                         | -                | -            |
| `Number.prototype.constructor`                              | -                | -            |
| `Number.prototype.toExponential()`                          | -                | -            |
| `Number.prototype.toFixed()`                                | -                | -            |
| `Number.prototype.toLocaleString()`                         | -                | -            |
| `Number.prototype.toPrecision()`                            | -                | -            |
| `Number.prototype.toString()`                               | -                | -            |
| `Number.prototype.valueOf()`                                | -                | -            |
| `BigInt`                                                    | -                | -            |
| `BigInt.BigInt()`                                           | -                | -            |
| `BigInt.asIntN()`                                           | -                | -            |
| `BigInt.asUintN()`                                          | -                | -            |
| `BigInt.prototype.constructor`                              | -                | -            |
| `BigInt.prototype[@@toStringTag]`                           | -                | -            |
| `BigInt.prototype.toLocaleString()`                         | -                | -            |
| `BigInt.prototype.toString()`                               | -                | -            |
| `BigInt.prototype.valueOf()`                                | -                | -            |
| `Math`                                                      | -                | -            |
| `Math[@@toStringTag]`                                       | -                | -            |
| `Math.E`                                                    | -                | -            |
| `Math.LN2`                                                  | -                | -            |
| `Math.LN10`                                                 | -                | -            |
| `Math.LOG2E`                                                | -                | -            |
| `Math.LOG10E`                                               | -                | -            |
| `Math.PI`                                                   | -                | -            |
| `Math.SQRT1_2`                                              | -                | -            |
| `Math.SQRT2`                                                | -                | -            |
| `Math.abs()`                                                | -                | -            |
| `Math.acos()`                                               | -                | -            |
| `Math.acosh()`                                              | -                | -            |
| `Math.asin()`                                               | -                | -            |
| `Math.asinh()`                                              | -                | -            |
| `Math.atan()`                                               | -                | -            |
| `Math.atanh()`                                              | -                | -            |
| `Math.atan2()`                                              | -                | -            |
| `Math.cbrt()`                                               | -                | -            |
| `Math.ceil()`                                               | -                | -            |
| `Math.clz32()`                                              | -                | -            |
| `Math.cos()`                                                | -                | -            |
| `Math.cosh()`                                               | -                | -            |
| `Math.exp()`                                                | -                | -            |
| `Math.expm1()`                                              | -                | -            |
| `Math.floor()`                                              | -                | -            |
| `Math.fround()`                                             | -                | -            |
| `Math.hypot()`                                              | -                | -            |
| `Math.imul()`                                               | -                | -            |
| `Math.log()`                                                | -                | -            |
| `Math.log1p()`                                              | -                | -            |
| `Math.log10()`                                              | -                | -            |
| `Math.log2()`                                               | -                | -            |
| `Math.max()`                                                | -                | -            |
| `Math.min()`                                                | -                | -            |
| `Math.pow()`                                                | -                | -            |
| `Math.random()`                                             | -                | -            |
| `Math.round()`                                              | -                | -            |
| `Math.sign()`                                               | -                | -            |
| `Math.sin()`                                                | -                | -            |
| `Math.sinh()`                                               | -                | -            |
| `Math.sqrt()`                                               | -                | -            |
| `Math.tan()`                                                | -                | -            |
| `Math.tanh()`                                               | -                | -            |
| `Math.trunc()`                                              | -                | -            |
| `Date`                                                      | -                | -            |
| `Date.parse()`                                              | -                | -            |
| `Date.UTC()`                                                | -                | -            |
| `Date.prototype.getDate()`                                  | -                | -            |
| `Date.prototype.getDay()`                                   | -                | -            |
| `Date.prototype.getFullYear()`                              | -                | -            |
| `Date.prototype.getHours()`                                 | -                | -            |
| `Date.prototype.getMilliseconds()`                          | -                | -            |
| `Date.prototype.getMinutes()`                               | -                | -            |
| `Date.prototype.getMonth()`                                 | -                | -            |
| `Date.prototype.getSeconds()`                               | -                | -            |
| `Date.prototype.getTime()`                                  | -                | -            |
| `Date.prototype.getTimezoneOffset()`                        | -                | -            |
| `Date.prototype.getUTCDate()`                               | -                | -            |
| `Date.prototype.getUTCDay()`                                | -                | -            |
| `Date.prototype.getUTCFullYear()`                           | -                | -            |
| `Date.prototype.getUTCHours()`                              | -                | -            |
| `Date.prototype.getUTCMilliseconds()`                       | -                | -            |
| `Date.prototype.getUTCMinutes()`                            | -                | -            |
| `Date.prototype.getUTCMonth()`                              | -                | -            |
| `Date.prototype.getUTCSeconds()`                            | -                | -            |
| `Date.prototype.setDate()`                                  | -                | -            |
| `Date.prototype.setFullYear()`                              | -                | -            |
| `Date.prototype.setHours()`                                 | -                | -            |
| `Date.prototype.setMilliseconds()`                          | -                | -            |
| `Date.prototype.setMinutes()`                               | -                | -            |
| `Date.prototype.setMonth()`                                 | -                | -            |
| `Date.prototype.setSeconds()`                               | -                | -            |
| `Date.prototype.setTime()`                                  | -                | -            |
| `Date.prototype.setUTCDate()`                               | -                | -            |
| `Date.prototype.setUTCFullYear()`                           | -                | -            |
| `Date.prototype.setUTCHours()`                              | -                | -            |
| `Date.prototype.setUTCMilliseconds()`                       | -                | -            |
| `Date.prototype.setUTCMinutes()`                            | -                | -            |
| `Date.prototype.setUTCMonth()`                              | -                | -            |
| `Date.prototype.setUTCSeconds()`                            | -                | -            |
| `Date.prototype.toDateString()`                             | -                | -            |
| `Date.prototype.toISOString()`                              | -                | -            |
| `Date.prototype.toJSON()`                                   | -                | -            |
| `Date.prototype.toLocaleDateString()`                       | -                | -            |
| `Date.prototype.toLocaleString()`                           | -                | -            |
| `Date.prototype.toLocaleTimeString()`                       | -                | -            |
| `Date.prototype.toString()`                                 | -                | -            |
| `Date.prototype.toTimeString()`                             | -                | -            |
| `Date.prototype.toUTCString()`                              | -                | -            |
| `Date.prototype.valueOf()`                                  | -                | -            |
| `Date.prototype[@@toPrimitive]()`                           | -                | -            |
| `String`                                                    | -                | -            |
| `String.String()`                                           | -                | -            |
| `String.fromCharCode()`                                     | -                | -            |
| `String.fromCodePoint()`                                    | -                | -            |
| `String.raw()`                                              | -                | -            |
| `String.prototype.constructor`                              | -                | -            |
| `String.length`                                             | -                | -            |
| `String.prototype.at()`                                     | -                | -            |
| `String.prototype.charAt()`                                 | -                | -            |
| `String.prototype.charCodeAt()`                             | -                | -            |
| `String.prototype.codePointAt()`                            | -                | -            |
| `String.prototype.concat()`                                 | -                | -            |
| `String.prototype.endsWith()`                               | -                | -            |
| `String.prototype.includes()`                               | -                | -            |
| `String.prototype.indexOf()`                                | -                | -            |
| `String.prototype.isWellFormed()`                           | -                | -            |
| `String.prototype.lastIndexOf()`                            | -                | -            |
| `String.prototype.localeCompare()`                          | -                | -            |
| `String.prototype.match()`                                  | -                | -            |
| `String.prototype.matchAll()`                               | -                | -            |
| `String.prototype.normalize()`                              | -                | -            |
| `String.prototype.padEnd()`                                 | -                | -            |
| `String.prototype.padStart()`                               | -                | -            |
| `String.prototype.repeat()`                                 | -                | -            |
| `String.prototype.replace()`                                | -                | -            |
| `String.prototype.replaceAll()`                             | -                | -            |
| `String.prototype.search()`                                 | -                | -            |
| `String.prototype.slice()`                                  | -                | -            |
| `String.prototype.split()`                                  | -                | -            |
| `String.prototype.startsWith()`                             | -                | -            |
| `String.prototype.substring()`                              | -                | -            |
| `String.prototype.toLocaleLowerCase()`                      | -                | -            |
| `String.prototype.toLocaleUpperCase()`                      | -                | -            |
| `String.prototype.toLowerCase()`                            | -                | -            |
| `String.prototype.toString()`                               | -                | -            |
| `String.prototype.toUpperCase()`                            | -                | -            |
| `String.prototype.toWellFormed()`                           | -                | -            |
| `String.prototype.trim()`                                   | -                | -            |
| `String.prototype.trimStart()`                              | -                | -            |
| `String.prototype.trimEnd()`                                | -                | -            |
| `String.prototype.valueOf()`                                | -                | -            |
| `String.prototype[@@iterator]()`                            | -                | -            |
| `RegExp`                                                    | -                | -            |
| `RegExp.RegExp()`                                           | -                | -            |
| `RegExp[@@species]`                                         | -                | -            |
| `RegExp.prototype.constructor`                              | -                | -            |
| `RegExp.prototype.dotAll`                                   | -                | -            |
| `RegExp.prototype.flags`                                    | -                | -            |
| `RegExp.prototype.global`                                   | -                | -            |
| `RegExp.prototype.hasIndices`                               | -                | -            |
| `RegExp.prototype.ignoreCase`                               | -                | -            |
| `RegExp.prototype.multiline`                                | -                | -            |
| `RegExp.prototype.source`                                   | -                | -            |
| `RegExp.prototype.sticky`                                   | -                | -            |
| `RegExp.prototype.unicode`                                  | -                | -            |
| `RegExp.lastIndex`                                          | -                | -            |
| `RegExp.prototype.exec()`                                   | -                | -            |
| `RegExp.prototype.test()`                                   | -                | -            |
| `RegExp.prototype.toString()`                               | -                | -            |
| `RegExp.prototype[@@match]()`                               | -                | -            |
| `RegExp.prototype[@@matchAll]()`                            | -                | -            |
| `RegExp.prototype[@@replace]()`                             | -                | -            |
| `RegExp.prototype[@@search]()`                              | -                | -            |
| `RegExp.prototype[@@split]()`                               | -                | -            |
| `Array`                                                     | -                | -            |
| `Array.Array()`                                             | -                | -            |
| `Array[@@species]`                                          | -                | -            |
| `Array.from()`                                              | -                | -            |
| `Array.isArray()`                                           | -                | -            |
| `Array.of()`                                                | -                | -            |
| `Array.prototype.constructor`                               | -                | -            |
| `Array.prototype[@@unscopables]`                            | -                | -            |
| `Array.length`                                              | -                | -            |
| `Array.prototype.at()`                                      | -                | -            |
| `Array.prototype.concat()`                                  | -                | -            |
| `Array.prototype.copyWithin()`                              | -                | -            |
| `Array.prototype.entries()`                                 | -                | -            |
| `Array.prototype.every()`                                   | -                | -            |
| `Array.prototype.fill()`                                    | -                | -            |
| `Array.prototype.filter()`                                  | -                | -            |
| `Array.prototype.find()`                                    | -                | -            |
| `Array.prototype.findIndex()`                               | -                | -            |
| `Array.prototype.findLast()`                                | -                | -            |
| `Array.prototype.findLastIndex()`                           | -                | -            |
| `Array.prototype.flat()`                                    | -                | -            |
| `Array.prototype.flatMap()`                                 | -                | -            |
| `Array.prototype.forEach()`                                 | -                | -            |
| `Array.prototype.group()`                                   | -                | -            |
| `Array.prototype.groupToMap()`                              | -                | -            |
| `Array.prototype.includes()`                                | -                | -            |
| `Array.prototype.indexOf()`                                 | -                | -            |
| `Array.prototype.join()`                                    | -                | -            |
| `Array.prototype.keys()`                                    | -                | -            |
| `Array.prototype.lastIndexOf()`                             | -                | -            |
| `Array.prototype.map()`                                     | -                | -            |
| `Array.prototype.pop()`                                     | -                | -            |
| `Array.prototype.push()`                                    | -                | -            |
| `Array.prototype.reduce()`                                  | -                | -            |
| `Array.prototype.reduceRight()`                             | -                | -            |
| `Array.prototype.reverse()`                                 | -                | -            |
| `Array.prototype.shift()`                                   | -                | -            |
| `Array.prototype.slice()`                                   | -                | -            |
| `Array.prototype.some()`                                    | -                | -            |
| `Array.prototype.sort()`                                    | -                | -            |
| `Array.prototype.splice()`                                  | -                | -            |
| `Array.prototype.toLocaleString()`                          | -                | -            |
| `Array.prototype.toReversed()`                              | -                | -            |
| `Array.prototype.toSorted()`                                | -                | -            |
| `Array.prototype.toSpliced()`                               | -                | -            |
| `Array.prototype.toString()`                                | -                | -            |
| `Array.prototype.unshift()`                                 | -                | -            |
| `Array.prototype.values()`                                  | -                | -            |
| `Array.prototype.with()`                                    | -                | -            |
| `Array.prototype[@@iterator]()`                             | -                | -            |
| `TypedArray` _(Hidden)_                                     | -                | -            |
| `TypedArray[@@species]`                                     | -                | -            |
| `TypedArray.BYTES_PER_ELEMENT`                              | -                | -            |
| `TypedArray.from()`                                         | -                | -            |
| `TypedArray.of()`                                           | -                | -            |
| `TypedArray.prototype.buffer`                               | -                | -            |
| `TypedArray.prototype.byteLength`                           | -                | -            |
| `TypedArray.prototype.byteOffset`                           | -                | -            |
| `TypedArray.prototype.constructor`                          | -                | -            |
| `TypedArray.prototype.length`                               | -                | -            |
| `TypedArray.prototype[@@toStringTag]`                       | -                | -            |
| `TypedArray.prototype.BYTES_PER_ELEMENT`                    | -                | -            |
| `TypedArray.prototype.at()`                                 | -                | -            |
| `TypedArray.prototype.copyWithin()`                         | -                | -            |
| `TypedArray.prototype.entries()`                            | -                | -            |
| `TypedArray.prototype.every()`                              | -                | -            |
| `TypedArray.prototype.fill()`                               | -                | -            |
| `TypedArray.prototype.filter()`                             | -                | -            |
| `TypedArray.prototype.find()`                               | -                | -            |
| `TypedArray.prototype.findIndex()`                          | -                | -            |
| `TypedArray.prototype.findLast()`                           | -                | -            |
| `TypedArray.prototype.findLastIndex()`                      | -                | -            |
| `TypedArray.prototype.forEach()`                            | -                | -            |
| `TypedArray.prototype.includes()`                           | -                | -            |
| `TypedArray.prototype.indexOf()`                            | -                | -            |
| `TypedArray.prototype.join()`                               | -                | -            |
| `TypedArray.prototype.keys()`                               | -                | -            |
| `TypedArray.prototype.lastIndexOf()`                        | -                | -            |
| `TypedArray.prototype.map()`                                | -                | -            |
| `TypedArray.prototype.reduce()`                             | -                | -            |
| `TypedArray.prototype.reduceRight()`                        | -                | -            |
| `TypedArray.prototype.reverse()`                            | -                | -            |
| `TypedArray.prototype.set()`                                | -                | -            |
| `TypedArray.prototype.slice()`                              | -                | -            |
| `TypedArray.prototype.some()`                               | -                | -            |
| `TypedArray.prototype.sort()`                               | -                | -            |
| `TypedArray.prototype.subarray()`                           | -                | -            |
| `TypedArray.prototype.toLocaleString()`                     | -                | -            |
| `TypedArray.prototype.toReversed()`                         | -                | -            |
| `TypedArray.prototype.toSorted()`                           | -                | -            |
| `TypedArray.prototype.toString()`                           | -                | -            |
| `TypedArray.prototype.values()`                             | -                | -            |
| `TypedArray.prototype.with()`                               | -                | -            |
| `TypedArray.prototype[@@iterator]()`                        | -                | -            |
| `Int8Array`                                                 | -                | -            |
| `Int8Array.Int8Array()`                                     | -                | -            |
| `Int8Array.BYTES_PER_ELEMENT`                               | -                | -            |
| `Int8Array.prototype.BYTES_PER_ELEMENT`                     | -                | -            |
| `Int8Array.prototype.constructor`                           | -                | -            |
| `Uint8Array`                                                | -                | -            |
| `Uint8Array.Uint8Array()`                                   | -                | -            |
| `Uint8Array.BYTES_PER_ELEMENT`                              | -                | -            |
| `Uint8Array.prototype.BYTES_PER_ELEMENT`                    | -                | -            |
| `Uint8Array.prototype.constructor`                          | -                | -            |
| `Uint8ClampedArray`                                         | -                | -            |
| `Uint8ClampedArray.Uint8ClampedArray()`                     | -                | -            |
| `Uint8ClampedArray.BYTES_PER_ELEMENT`                       | -                | -            |
| `Uint8ClampedArray.prototype.BYTES_PER_ELEMENT`             | -                | -            |
| `Uint8ClampedArray.prototype.constructor`                   | -                | -            |
| `Int16Array`                                                | -                | -            |
| `Int16Array.Int16Array()`                                   | -                | -            |
| `Int16Array.BYTES_PER_ELEMENT`                              | -                | -            |
| `Int16Array.prototype.BYTES_PER_ELEMENT`                    | -                | -            |
| `Int16Array.prototype.constructor`                          | -                | -            |
| `Uint16Array`                                               | -                | -            |
| `Uint16Array.Uint16Array()`                                 | -                | -            |
| `Uint16Array.BYTES_PER_ELEMENT`                             | -                | -            |
| `Uint16Array.prototype.BYTES_PER_ELEMENT`                   | -                | -            |
| `Uint16Array.prototype.constructor`                         | -                | -            |
| `Int32Array`                                                | -                | -            |
| `Int32Array.Int32Array()`                                   | -                | -            |
| `Int32Array.BYTES_PER_ELEMENT`                              | -                | -            |
| `Int32Array.prototype.BYTES_PER_ELEMENT`                    | -                | -            |
| `Int32Array.prototype.constructor`                          | -                | -            |
| `Uint32Array`                                               | -                | -            |
| `Uint32Array.Uint32Array()`                                 | -                | -            |
| `Uint32Array.BYTES_PER_ELEMENT`                             | -                | -            |
| `Uint32Array.prototype.BYTES_PER_ELEMENT`                   | -                | -            |
| `Uint32Array.prototype.constructor`                         | -                | -            |
| `BigInt64Array`                                             | -                | -            |
| `BigInt64Array.BigInt64Array()`                             | -                | -            |
| `BigInt64Array.BYTES_PER_ELEMENT`                           | -                | -            |
| `BigInt64Array.prototype.BYTES_PER_ELEMENT`                 | -                | -            |
| `BigInt64Array.prototype.constructor`                       | -                | -            |
| `BigUint64Array`                                            | -                | -            |
| `BigUint64Array.BigUint64Array()`                           | -                | -            |
| `BigUint64Array.BYTES_PER_ELEMENT`                          | -                | -            |
| `BigUint64Array.prototype.BYTES_PER_ELEMENT`                | -                | -            |
| `BigUint64Array.prototype.constructor`                      | -                | -            |
| `Float32Array`                                              | -                | -            |
| `Float32Array.Float32Array()`                               | -                | -            |
| `Float32Array.BYTES_PER_ELEMENT`                            | -                | -            |
| `Float32Array.prototype.BYTES_PER_ELEMENT`                  | -                | -            |
| `Float32Array.prototype.constructor`                        | -                | -            |
| `Float64Array`                                              | -                | -            |
| `Float64Array.Float64Array()`                               | -                | -            |
| `Float64Array.BYTES_PER_ELEMENT`                            | -                | -            |
| `Float64Array.prototype.BYTES_PER_ELEMENT`                  | -                | -            |
| `Float64Array.prototype.constructor`                        | -                | -            |
| `Map`                                                       | -                | -            |
| `Map.Map()`                                                 | -                | -            |
| `Map[@@species]`                                            | -                | -            |
| `Map.prototype.constructor`                                 | -                | -            |
| `Map.prototype.size`                                        | -                | -            |
| `Map.prototype[@@toStringTag]`                              | -                | -            |
| `Map.prototype.clear()`                                     | -                | -            |
| `Map.prototype.delete()`                                    | -                | -            |
| `Map.prototype.entries()`                                   | -                | -            |
| `Map.prototype.forEach()`                                   | -                | -            |
| `Map.prototype.get()`                                       | -                | -            |
| `Map.prototype.has()`                                       | -                | -            |
| `Map.prototype.keys()`                                      | -                | -            |
| `Map.prototype.set()`                                       | -                | -            |
| `Map.prototype.values()`                                    | -                | -            |
| `Map.prototype[@@iterator]()`                               | -                | -            |
| `Set`                                                       | -                | -            |
| `Set.Set()`                                                 | -                | -            |
| `Set[@@species]`                                            | -                | -            |
| `Set.prototype.constructor`                                 | -                | -            |
| `Set.prototype.size`                                        | -                | -            |
| `Set.prototype[@@toStringTag]`                              | -                | -            |
| `Set.prototype.add()`                                       | -                | -            |
| `Set.prototype.clear()`                                     | -                | -            |
| `Set.prototype.delete()`                                    | -                | -            |
| `Set.prototype.entries()`                                   | -                | -            |
| `Set.prototype.forEach()`                                   | -                | -            |
| `Set.prototype.has()`                                       | -                | -            |
| `Set.prototype.keys()`                                      | -                | -            |
| `Set.prototype.values()`                                    | -                | -            |
| `Set.prototype[@@iterator]()`                               | -                | -            |
| `WeakMap`                                                   | -                | -            |
| `WeakMap.WeakMap()`                                         | -                | -            |
| `WeakMap.prototype.constructor`                             | -                | -            |
| `WeakMap.prototype[@@toStringTag]`                          | -                | -            |
| `WeakMap.prototype.delete()`                                | -                | -            |
| `WeakMap.prototype.get()`                                   | -                | -            |
| `WeakMap.prototype.has()`                                   | -                | -            |
| `WeakMap.prototype.set()`                                   | -                | -            |
| `WeakSet`                                                   | -                | -            |
| `WeakSet.WeakSet()`                                         | -                | -            |
| `WeakSet.prototype.constructor`                             | -                | -            |
| `WeakSet.prototype[@@toStringTag]`                          | -                | -            |
| `WeakSet.prototype.add()`                                   | -                | -            |
| `WeakSet.prototype.delete()`                                | -                | -            |
| `WeakSet.prototype.has()`                                   | -                | -            |
| `ArrayBuffer`                                               | -                | -            |
| `ArrayBuffer.ArrayBuffer()`                                 | -                | -            |
| `ArrayBuffer[@@species]`                                    | -                | -            |
| `ArrayBuffer.isView()`                                      | -                | -            |
| `ArrayBuffer.prototype.byteLength`                          | -                | -            |
| `ArrayBuffer.prototype.constructor`                         | -                | -            |
| `ArrayBuffer.prototype.maxByteLength`                       | -                | -            |
| `ArrayBuffer.prototype.resizable`                           | -                | -            |
| `ArrayBuffer.prototype[@@toStringTag]`                      | -                | -            |
| `ArrayBuffer.prototype.resize()`                            | -                | -            |
| `ArrayBuffer.prototype.slice()`                             | -                | -            |
| `DataView`                                                  | -                | -            |
| `DataView.DataView()`                                       | -                | -            |
| `DataView.prototype.buffer`                                 | -                | -            |
| `DataView.prototype.byteLength`                             | -                | -            |
| `DataView.prototype.byteOffset`                             | -                | -            |
| `DataView.prototype.constructor`                            | -                | -            |
| `DataView.prototype[@@toStringTag]`                         | -                | -            |
| `DataView.prototype.getBigInt64()`                          | -                | -            |
| `DataView.prototype.getBigUint64()`                         | -                | -            |
| `DataView.prototype.getFloat32()`                           | -                | -            |
| `DataView.prototype.getFloat64()`                           | -                | -            |
| `DataView.prototype.getInt16()`                             | -                | -            |
| `DataView.prototype.getInt32()`                             | -                | -            |
| `DataView.prototype.getInt8()`                              | -                | -            |
| `DataView.prototype.getUint16()`                            | -                | -            |
| `DataView.prototype.getUint32()`                            | -                | -            |
| `DataView.prototype.getUint8()`                             | -                | -            |
| `DataView.prototype.setBigInt64()`                          | -                | -            |
| `DataView.prototype.setBigUint64()`                         | -                | -            |
| `DataView.prototype.setFloat32()`                           | -                | -            |
| `DataView.prototype.setFloat64()`                           | -                | -            |
| `DataView.prototype.setInt16()`                             | -                | -            |
| `DataView.prototype.setInt32()`                             | -                | -            |
| `DataView.prototype.setInt8()`                              | -                | -            |
| `DataView.prototype.setUint16()`                            | -                | -            |
| `DataView.prototype.setUint32()`                            | -                | -            |
| `DataView.prototype.setUint8()`                             | -                | -            |
| `JSON`                                                      | -                | -            |
| `JSON[@@toStringTag]`                                       | -                | -            |
| `JSON.parse()`                                              | -                | -            |
| `JSON.stringify()`                                          | -                | -            |
| `WeakRef`                                                   | -                | -            |
| `WeakRef.WeakRef()`                                         | -                | -            |
| `WeakRef.prototype[@@toStringTag]`                          | -                | -            |
| `Iterator`                                                  | -                | -            |
| `Iterator.prototype[@@iterator]()`                          | -                | -            |
| `GeneratorFunction`                                         | -                | -            |
| `GeneratorFunction.GeneratorFunction()`                     | -                | -            |
| `GeneratorFunction.prototype.constructor`                   | -                | -            |
| `GeneratorFunction.prototype.prototype`                     | -                | -            |
| `GeneratorFunction.prototype[@@toStringTag]`                | -                | -            |
| `Generator`                                                 | -                | -            |
| `Generator.prototype.constructor`                           | -                | -            |
| `Generator.prototype[@@toStringTag]`                        | -                | -            |
| `Generator.prototype.next()`                                | -                | -            |
| `Generator.prototype.return()`                              | -                | -            |
| `Generator.prototype.throw()`                               | -                | -            |
| `Intl`                                                      | -                | -            |
| `Intl.Collator()`                                           | -                | -            |
| `Intl.DateTimeFormat()`                                     | -                | -            |
| `Intl.DisplayNames()`                                       | -                | -            |
| `Intl.DurationFormat()`                                     | -                | -            |
| `Intl.ListFormat()`                                         | -                | -            |
| `Intl.Locale()`                                             | -                | -            |
| `Intl.NumberFormat()`                                       | -                | -            |
| `Intl.PluralRules()`                                        | -                | -            |
| `Intl.RelativeTimeFormat()`                                 | -                | -            |
| `Intl.Segmenter()`                                          | -                | -            |
| `Intl.getCanonicalLocales()`                                | -                | -            |
| `Intl.supportedValuesOf()`                                  | -                | -            |
| `Intl.Collator`                                             | -                | -            |
| `Intl.Collator.Collator()`                                  | -                | -            |
| `Intl.Collator.supportedLocalesOf()`                        | -                | -            |
| `Intl.Collator.prototype.constructor`                       | -                | -            |
| `Intl.Collator.prototype[@@toStringTag]`                    | -                | -            |
| `Intl.Collator.prototype.compare()`                         | -                | -            |
| `Intl.Collator.prototype.resolvedOptions()`                 | -                | -            |
| `Intl.DateTimeFormat`                                       | -                | -            |
| `Intl.DateTimeFormat.DateTimeFormat()`                      | -                | -            |
| `Intl.DateTimeFormat.supportedLocalesOf()`                  | -                | -            |
| `Intl.DateTimeFormat.prototype.constructor`                 | -                | -            |
| `Intl.DateTimeFormat.prototype[@@toStringTag]`              | -                | -            |
| `Intl.DateTimeFormat.prototype.format()`                    | -                | -            |
| `Intl.DateTimeFormat.prototype.formatRange()`               | -                | -            |
| `Intl.DateTimeFormat.prototype.formatRangeToParts()`        | -                | -            |
| `Intl.DateTimeFormat.prototype.formatToParts()`             | -                | -            |
| `Intl.DateTimeFormat.prototype.resolvedOptions()`           | -                | -            |
| `Intl.DisplayNames`                                         | -                | -            |
| `Intl.DisplayNames.DisplayNames()`                          | -                | -            |
| `Intl.DisplayNames.supportedLocalesOf()`                    | -                | -            |
| `Intl.DisplayNames.prototype.constructor`                   | -                | -            |
| `Intl.DisplayNames.prototype[@@toStringTag]`                | -                | -            |
| `Intl.DisplayNames.prototype.of()`                          | -                | -            |
| `Intl.DisplayNames.prototype.resolvedOptions()`             | -                | -            |
| `Intl.ListFormat`                                           | -                | -            |
| `Intl.ListFormat.ListFormat()`                              | -                | -            |
| `Intl.ListFormat.supportedLocalesOf()`                      | -                | -            |
| `Intl.ListFormat.prototype.constructor`                     | -                | -            |
| `Intl.ListFormat.prototype[@@toStringTag]`                  | -                | -            |
| `Intl.ListFormat.prototype.format()`                        | -                | -            |
| `Intl.ListFormat.prototype.formatToParts()`                 | -                | -            |
| `Intl.ListFormat.prototype.resolvedOptions()`               | -                | -            |
| `Intl.Locale`                                               | -                | -            |
| `Intl.Locale.Locale()`                                      | -                | -            |
| `Intl.Locale.prototype.baseName`                            | -                | -            |
| `Intl.Locale.prototype.calendar`                            | -                | -            |
| `Intl.Locale.prototype.calendars`                           | -                | -            |
| `Intl.Locale.prototype.caseFirst`                           | -                | -            |
| `Intl.Locale.prototype.collation`                           | -                | -            |
| `Intl.Locale.prototype.collations`                          | -                | -            |
| `Intl.Locale.prototype.constructor`                         | -                | -            |
| `Intl.Locale.prototype.hourCycle`                           | -                | -            |
| `Intl.Locale.prototype.hourCycles`                          | -                | -            |
| `Intl.Locale.prototype.language`                            | -                | -            |
| `Intl.Locale.prototype.numberingSystem`                     | -                | -            |
| `Intl.Locale.prototype.numberingSystems`                    | -                | -            |
| `Intl.Locale.prototype.numeric`                             | -                | -            |
| `Intl.Locale.prototype.region`                              | -                | -            |
| `Intl.Locale.prototype.script`                              | -                | -            |
| `Intl.Locale.prototype.textInfo`                            | -                | -            |
| `Intl.Locale.prototype.timeZones`                           | -                | -            |
| `Intl.Locale.prototype.weekInfo`                            | -                | -            |
| `Intl.Locale.prototype[@@toStringTag]`                      | -                | -            |
| `Intl.Locale.prototype.maximize()`                          | -                | -            |
| `Intl.Locale.prototype.minimize()`                          | -                | -            |
| `Intl.Locale.prototype.toString()`                          | -                | -            |
| `Intl.NumberFormat`                                         | -                | -            |
| `Intl.NumberFormat.NumberFormat()`                          | -                | -            |
| `Intl.NumberFormat.supportedLocalesOf()`                    | -                | -            |
| `Intl.NumberFormat.prototype.constructor`                   | -                | -            |
| `Intl.NumberFormat.prototype[@@toStringTag]`                | -                | -            |
| `Intl.NumberFormat.prototype.format()`                      | -                | -            |
| `Intl.NumberFormat.prototype.formatRange()`                 | -                | -            |
| `Intl.NumberFormat.prototype.formatRangeToParts()`          | -                | -            |
| `Intl.NumberFormat.prototype.formatToParts()`               | -                | -            |
| `Intl.NumberFormat.prototype.resolvedOptions()`             | -                | -            |
| `Intl.PluralRules`                                          | -                | -            |
| `Intl.PluralRules.PluralRules()`                            | -                | -            |
| `Intl.PluralRules.supportedLocalesOf()`                     | -                | -            |
| `Intl.PluralRules.prototype.constructor`                    | -                | -            |
| `Intl.PluralRules.prototype[@@toStringTag]`                 | -                | -            |
| `Intl.PluralRules.prototype.resolvedOptions()`              | -                | -            |
| `Intl.PluralRules.prototype.select()`                       | -                | -            |
| `Intl.PluralRules.prototype.selectRange()`                  | -                | -            |
| `Intl.RelativeTimeFormat`                                   | -                | -            |
| `Intl.RelativeTimeFormat.RelativeTimeFormat()`              | -                | -            |
| `Intl.RelativeTimeFormat.supportedLocalesOf()`              | -                | -            |
| `Intl.RelativeTimeFormat.prototype.constructor`             | -                | -            |
| `Intl.RelativeTimeFormat.prototype[@@toStringTag]`          | -                | -            |
| `Intl.RelativeTimeFormat.prototype.format()`                | -                | -            |
| `Intl.RelativeTimeFormat.prototype.formatToParts()`         | -                | -            |
| `Intl.RelativeTimeFormat.prototype.resolvedOptions()`       | -                | -            |
| `Intl.Segmenter`                                            | -                | -            |
| `Intl.Segmenter.Segmenter()`                                | -                | -            |
| `Intl.Segmenter.supportedLocalesOf()`                       | -                | -            |
| `Intl.Segmenter.prototype.constructor`                      | -                | -            |
| `Intl.Segmenter.prototype[@@toStringTag]`                   | -                | -            |
| `Intl.Segmenter.prototype.resolvedOptions()`                | -                | -            |
| `Intl.Segmenter.prototype.segment()`                        | -                | -            |
| `Array.fromAsync()`                                         | `Async`          | -            |
| `AsyncFunction`[^async-function]                            | `Async`          | -            |
| `AsyncFunction.AsyncFunction()`                             | `Async`          | -            |
| `AsyncFunction.prototype.constructor`                       | `Async`          | -            |
| `AsyncFunction.prototype[@@toStringTag]`                    | `Async`          | -            |
| `AsyncGenerator`                                            | `Async`          | -            |
| `AsyncGenerator.prototype.constructor`                      | `Async`          | -            |
| `AsyncGenerator.prototype[@@toStringTag]`                   | `Async`          | -            |
| `AsyncGenerator.prototype.next()`                           | `Async`          | -            |
| `AsyncGenerator.prototype.return()`                         | `Async`          | -            |
| `AsyncGenerator.prototype.throw()`                          | `Async`          | -            |
| `AsyncGeneratorFunction`                                    | `Async`          | -            |
| `AsyncGeneratorFunction.AsyncGeneratorFunction()`           | `Async`          | -            |
| `AsyncGeneratorFunction.prototype.constructor`              | `Async`          | -            |
| `AsyncGeneratorFunction.prototype.prototype`                | `Async`          | -            |
| `AsyncGeneratorFunction.prototype[@@toStringTag]`           | `Async`          | -            |
| `AsyncIterator`                                             | `Async`          | -            |
| `AsyncIterator.prototype[@@asyncIterator]()`                | `Async`          | -            |
| `Atomics`                                                   | `Async`          | -            |
| `Atomics[@@toStringTag]`                                    | `Async`          | -            |
| `Atomics.add()`                                             | `Async`          | -            |
| `Atomics.and()`                                             | `Async`          | -            |
| `Atomics.compareExchange()`                                 | `Async`          | -            |
| `Atomics.exchange()`                                        | `Async`          | -            |
| `Atomics.isLockFree(size)`                                  | `Async`          | -            |
| `Atomics.load()`                                            | `Async`          | -            |
| `Atomics.notify()`                                          | `Async`          | -            |
| `Atomics.or()`                                              | `Async`          | -            |
| `Atomics.store()`                                           | `Async`          | -            |
| `Atomics.sub()`                                             | `Async`          | -            |
| `Atomics.wait()`                                            | `Async`          | -            |
| `Atomics.waitAsync()`                                       | `Async`          | -            |
| `Atomics.xor()`                                             | `Async`          | -            |
| `Promise`                                                   | `Async`          | -            |
| `Promise.Promise()`                                         | `Async`          | -            |
| `Promise[@@species]`                                        | `Async`          | -            |
| `Promise.all()`                                             | `Async`          | -            |
| `Promise.allSettled()`                                      | `Async`          | -            |
| `Promise.any()`                                             | `Async`          | -            |
| `Promise.race()`                                            | `Async`          | -            |
| `Promise.reject()`                                          | `Async`          | -            |
| `Promise.resolve()`                                         | `Async`          | -            |
| `Promise.prototype.constructor`                             | `Async`          | -            |
| `Promise.prototype[@@toStringTag]`                          | `Async`          | -            |
| `Promise.prototype.catch()`                                 | `Async`          | -            |
| `Promise.prototype.finally()`                               | `Async`          | -            |
| `Promise.prototype.then()`                                  | `Async`          | -            |
| `SharedArrayBuffer`                                         | `Async`          | -            |
| `SharedArrayBuffer.SharedArrayBuffer()`                     | `Async`          | -            |
| `SharedArrayBuffer[@@species]`                              | `Async`          | -            |
| `SharedArrayBuffer.prototype.byteLength`                    | `Async`          | -            |
| `SharedArrayBuffer.prototype.constructor`                   | `Async`          | -            |
| `SharedArrayBuffer.prototype.growable`                      | `Async`          | -            |
| `SharedArrayBuffer.prototype.maxByteLength`                 | `Async`          | -            |
| `SharedArrayBuffer.prototype[@@toStringTag]`                | `Async`          | -            |
| `SharedArrayBuffer.prototype.grow()`                        | `Async`          | -            |
| `SharedArrayBuffer.prototype.slice()`                       | `Async`          | -            |
| `Symbol.asyncIterator`                                      | `Async`          | -            |
| `Date.Date()`                                               | `Date`           | -            |
| `Date.now()`                                                | `Date`           | -            |
| `Date.prototype.constructor`                                | `Date`           | -            |
| `ReadableStream`                                            | `Streams`        | `Async`      |
| `ReadableStream.ReadableStream()`                           | `Streams`        | `Async`      |
| `ReadableStream.locked`                                     | `Streams`        | `Async`      |
| `ReadableStream.cancel()`                                   | `Streams`        | `Async`      |
| `ReadableStream.getReader()`                                | `Streams`        | `Async`      |
| `ReadableStream.pipeThrough()`                              | `Streams`        | `Async`      |
| `ReadableStream.pipeTo()`                                   | `Streams`        | `Async`      |
| `ReadableStream.tee()`                                      | `Streams`        | `Async`      |
| `ReadableStreamDefaultReader`                               | `Streams`        | `Async`      |
| `ReadableStreamDefaultReader.ReadableStreamDefaultReader()` | `Streams`        | `Async`      |
| `ReadableStreamDefaultReader.closed`                        | `Streams`        | `Async`      |
| `ReadableStreamDefaultReader.cancel()`                      | `Streams`        | `Async`      |
| `ReadableStreamDefaultReader.read()`                        | `Streams`        | `Async`      |
| `ReadableStreamDefaultReader.releaseLock()`                 | `Streams`        | `Async`      |
| `ReadableStreamDefaultController`                           | `Streams`        | `Async`      |
| `ReadableStreamDefaultController.desiredSize`               | `Streams`        | `Async`      |
| `ReadableStreamDefaultController.close()`                   | `Streams`        | `Async`      |
| `ReadableStreamDefaultController.enqueue()`                 | `Streams`        | `Async`      |
| `ReadableStreamDefaultController.error()`                   | `Streams`        | `Async`      |
| `WritableStream`                                            | `Streams`        | `Async`      |
| `WritableStream.WritableStream()`                           | `Streams`        | `Async`      |
| `WritableStream.locked`                                     | `Streams`        | `Async`      |
| `WritableStream.abort()`                                    | `Streams`        | `Async`      |
| `WritableStream.close()`                                    | `Streams`        | `Async`      |
| `WritableStream.getWriter()`                                | `Streams`        | `Async`      |
| `WritableStreamDefaultWriter`                               | `Streams`        | `Async`      |
| `WritableStreamDefaultWriter.WritableStreamDefaultWriter()` | `Streams`        | `Async`      |
| `WritableStreamDefaultWriter.closed`                        | `Streams`        | `Async`      |
| `WritableStreamDefaultWriter.desiredSize`                   | `Streams`        | `Async`      |
| `WritableStreamDefaultWriter.ready`                         | `Streams`        | `Async`      |
| `WritableStreamDefaultWriter.abort()`                       | `Streams`        | `Async`      |
| `WritableStreamDefaultWriter.close()`                       | `Streams`        | `Async`      |
| `WritableStreamDefaultWriter.releaseLock()`                 | `Streams`        | `Async`      |
| `WritableStreamDefaultWriter.write()`                       | `Streams`        | `Async`      |
| `WritableStreamDefaultController`                           | `Streams`        | `Async`      |
| `WritableStreamDefaultController.signal`                    | `Streams`        | `Async`      |
| `WritableStreamDefaultController.error()`                   | `Streams`        | `Async`      |
| `TransformStream`                                           | `Streams`        | `Async`      |
| `TransformStream.TransformStream()`                         | `Streams`        | `Async`      |
| `TransformStream.readable`                                  | `Streams`        | `Async`      |
| `TransformStream.writable`                                  | `Streams`        | `Async`      |
| `TransformStreamDefaultController`                          | `Streams`        | `Async`      |
| `TransformStreamDefaultController.desiredSize`              | `Streams`        | `Async`      |
| `TransformStreamDefaultController.enqueue()`                | `Streams`        | `Async`      |
| `TransformStreamDefaultController.error()`                  | `Streams`        | `Async`      |
| `TransformStreamDefaultController.terminate()`              | `Streams`        | `Async`      |
| `ByteLengthQueuingStrategy`                                 | `Streams`        | `Async`      |
| `ByteLengthQueuingStrategy.ByteLengthQueuingStrategy()`     | `Streams`        | `Async`      |
| `ByteLengthQueuingStrategy.highWaterMark`                   | `Streams`        | `Async`      |
| `ByteLengthQueuingStrategy.size()`                          | `Streams`        | `Async`      |
| `CountQueuingStrategy`                                      | `Streams`        | `Async`      |
| `CountQueuingStrategy.CountQueuingStrategy()`               | `Streams`        | `Async`      |
| `CountQueuingStrategy.highWaterMark`                        | `Streams`        | `Async`      |
| `CountQueuingStrategy.size()`                               | `Streams`        | `Async`      |
| `ReadableStreamBYOBReader`                                  | `Streams`        | `Async`      |
| `ReadableStreamBYOBReader.ReadableStreamBYOBReader()`       | `Streams`        | `Async`      |
| `ReadableStreamBYOBReader.closed`                           | `Streams`        | `Async`      |
| `ReadableStreamBYOBReader.cancel()`                         | `Streams`        | `Async`      |
| `ReadableStreamBYOBReader.read()`                           | `Streams`        | `Async`      |
| `ReadableStreamBYOBReader.releaseLock()`                    | `Streams`        | `Async`      |
| `ReadableByteStreamController`                              | `Streams`        | `Async`      |
| `ReadableByteStreamController.byobRequest`                  | `Streams`        | `Async`      |
| `ReadableByteStreamController.desiredSize`                  | `Streams`        | `Async`      |
| `ReadableByteStreamController.close()`                      | `Streams`        | `Async`      |
| `ReadableByteStreamController.enqueue()`                    | `Streams`        | `Async`      |
| `ReadableByteStreamController.error()`                      | `Streams`        | `Async`      |
| `ReadableStreamBYOBRequest`                                 | `Streams`        | `Async`      |
| `ReadableStreamBYOBRequest.view`                            | `Streams`        | `Async`      |
| `ReadableStreamBYOBRequest.respond()`                       | `Streams`        | `Async`      |
| `ReadableStreamBYOBRequest.respondWithNewView()`            | `Streams`        | `Async`      |
| `TextDecoder`                                               | `Encoding`       | -            |
| `TextDecoder.TextDecoder()`                                 | `Encoding`       | -            |
| `TextDecoder.encoding`                                      | `Encoding`       | -            |
| `TextDecoder.fatal`                                         | `Encoding`       | -            |
| `TextDecoder.ignoreBOM`                                     | `Encoding`       | -            |
| `TextDecoder.decode()`                                      | `Encoding`       | -            |
| `TextEncoder`                                               | `Encoding`       | -            |
| `TextEncoder.TextEncoder()`                                 | `Encoding`       | -            |
| `TextEncoder.prototype.encoding`                            | `Encoding`       | -            |
| `TextEncoder.encode()`                                      | `Encoding`       | -            |
| `TextEncoder.encodeInto()`                                  | `Encoding`       | -            |
| `TextDecoderStream`                                         | `Encoding`       | `Streams`    |
| `TextDecoderStream.TextDecoderStream()`                     | `Encoding`       | `Streams`    |
| `TextDecoderStream.encoding`                                | `Encoding`       | `Streams`    |
| `TextDecoderStream.fatal`                                   | `Encoding`       | `Streams`    |
| `TextDecoderStream.ignoreBOM`                               | `Encoding`       | `Streams`    |
| `TextDecoderStream.readable`                                | `Encoding`       | `Streams`    |
| `TextDecoderStream.writable`                                | `Encoding`       | `Streams`    |
| `TextEncoderStream`                                         | `Encoding`       | `Streams`    |
| `TextEncoderStream.TextEncoderStream()`                     | `Encoding`       | `Streams`    |
| `TextEncoderStream.encoding`                                | `Encoding`       | `Streams`    |
| `TextEncoderStream.readable`                                | `Encoding`       | `Streams`    |
| `TextEncoderStream.writable`                                | `Encoding`       | `Streams`    |
| `CompressionStream`                                         | `Encoding`       | `Streams`    |
| `CompressionStream.CompressionStream()`                     | `Encoding`       | `Streams`    |
| `CompressionStream.readable`                                | `Encoding`       | `Streams`    |
| `CompressionStream.writable`                                | `Encoding`       | `Streams`    |
| `DecompressionStream`                                       | `Encoding`       | `Streams`    |
| `DecompressionStream.DecompressionStream()`                 | `Encoding`       | `Streams`    |
| `DecompressionStream.readable`                              | `Encoding`       | `Streams`    |
| `DecompressionStream.writable`                              | `Encoding`       | `Streams`    |
| `URL`                                                       | `URL`            | -            |
| `URL.URL()`                                                 | `URL`            | -            |
| `URL.hash`                                                  | `URL`            | -            |
| `URL.host`                                                  | `URL`            | -            |
| `URL.hostname`                                              | `URL`            | -            |
| `URL.href`                                                  | `URL`            | -            |
| `URL.origin`                                                | `URL`            | -            |
| `URL.password`                                              | `URL`            | -            |
| `URL.pathname`                                              | `URL`            | -            |
| `URL.port`                                                  | `URL`            | -            |
| `URL.protocol`                                              | `URL`            | -            |
| `URL.search`                                                | `URL`            | -            |
| `URL.searchParams`                                          | `URL`            | -            |
| `URL.username`                                              | `URL`            | -            |
| `URL.createObjectURL()`                                     | `URL`            | -            |
| `URL.revokeObjectURL()`                                     | `URL`            | -            |
| `URL.toString()`                                            | `URL`            | -            |
| `URL.toJSON()`                                              | `URL`            | -            |
| `URLSearchParams`                                           | `URL`            | -            |
| `URLSearchParams.URLSearchParams()`                         | `URL`            | -            |
| `URLSearchParams.append()`                                  | `URL`            | -            |
| `URLSearchParams.delete()`                                  | `URL`            | -            |
| `URLSearchParams.entries()`                                 | `URL`            | -            |
| `URLSearchParams.forEach()`                                 | `URL`            | -            |
| `URLSearchParams.get()`                                     | `URL`            | -            |
| `URLSearchParams.getAll()`                                  | `URL`            | -            |
| `URLSearchParams.has()`                                     | `URL`            | -            |
| `URLSearchParams.keys()`                                    | `URL`            | -            |
| `URLSearchParams.set()`                                     | `URL`            | -            |
| `URLSearchParams.sort()`                                    | `URL`            | -            |
| `URLSearchParams.toString()`                                | `URL`            | -            |
| `URLSearchParams.values()`                                  | `URL`            | -            |
| `URLPattern`                                                | `URLPattern`     | -            |
| `URLPattern.URLPattern()`                                   | `URLPattern`     | -            |
| `URLPattern.hash`                                           | `URLPattern`     | -            |
| `URLPattern.hostname`                                       | `URLPattern`     | -            |
| `URLPattern.password`                                       | `URLPattern`     | -            |
| `URLPattern.pathname`                                       | `URLPattern`     | -            |
| `URLPattern.port`                                           | `URLPattern`     | -            |
| `URLPattern.protocol`                                       | `URLPattern`     | -            |
| `URLPattern.search`                                         | `URLPattern`     | -            |
| `URLPattern.username`                                       | `URLPattern`     | -            |
| `URLPattern.exec()`                                         | `URLPattern`     | -            |
| `URLPattern.test()`                                         | `URLPattern`     | -            |
| `crypto`                                                    | `Crypto`         | `Async`      |
| `Crypto`                                                    | `Crypto`         | `Async`      |
| `Crypto.subtle`                                             | `Crypto`         | `Async`      |
| `Crypto.getRandomValues()`                                  | `Crypto`         | `Async`      |
| `Crypto.randomUUID()`                                       | `Crypto`         | `Async`      |
| `CryptoKey`                                                 | `Crypto`         | `Async`      |
| `CryptoKey.type`                                            | `Crypto`         | `Async`      |
| `CryptoKey.extractable`                                     | `Crypto`         | `Async`      |
| `CryptoKey.algorithm`                                       | `Crypto`         | `Async`      |
| `CryptoKey.usages`                                          | `Crypto`         | `Async`      |
| `SubtleCrypto`                                              | `Crypto`         | `Async`      |
| `SubtleCrypto.encrypt()`                                    | `Crypto`         | `Async`      |
| `SubtleCrypto.decrypt()`                                    | `Crypto`         | `Async`      |
| `SubtleCrypto.sign()`                                       | `Crypto`         | `Async`      |
| `SubtleCrypto.verify()`                                     | `Crypto`         | `Async`      |
| `SubtleCrypto.digest()`                                     | `Crypto`         | `Async`      |
| `SubtleCrypto.generateKey()`                                | `Crypto`         | `Async`      |
| `SubtleCrypto.deriveKey()`                                  | `Crypto`         | `Async`      |
| `SubtleCrypto.deriveBits()`                                 | `Crypto`         | `Async`      |
| `SubtleCrypto.importKey()`                                  | `Crypto`         | `Async`      |
| `SubtleCrypto.exportKey()`                                  | `Crypto`         | `Async`      |
| `SubtleCrypto.wrapKey()`                                    | `Crypto`         | `Async`      |
| `SubtleCrypto.unwrapKey()`                                  | `Crypto`         | `Async`      |
| `FormData`                                                  | `XMLHttpRequest` | `Async`      |
| `FormData.FormData()`                                       | `XMLHttpRequest` | `Async`      |
| `FormData.append()`                                         | `XMLHttpRequest` | `Async`      |
| `FormData.delete()`                                         | `XMLHttpRequest` | `Async`      |
| `FormData.entries()`                                        | `XMLHttpRequest` | `Async`      |
| `FormData.get()`                                            | `XMLHttpRequest` | `Async`      |
| `FormData.getAll()`                                         | `XMLHttpRequest` | `Async`      |
| `FormData.has()`                                            | `XMLHttpRequest` | `Async`      |
| `FormData.keys()`                                           | `XMLHttpRequest` | `Async`      |
| `FormData.set()`                                            | `XMLHttpRequest` | `Async`      |
| `FormData.values()`                                         | `XMLHttpRequest` | `Async`      |
| `ProgressEvent`                                             | `XMLHttpRequest` | `Async`      |
| `ProgressEvent.ProgressEvent()`                             | `XMLHttpRequest` | `Async`      |
| `ProgressEvent.lengthComputable`                            | `XMLHttpRequest` | `Async`      |
| `ProgressEvent.loaded`                                      | `XMLHttpRequest` | `Async`      |
| `ProgressEvent.total`                                       | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest`                                            | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.XMLHttpRequest()`                           | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.readyState`                                 | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.response`                                   | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.responseText`                               | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.responseType`                               | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.responseURL`                                | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.responseXML`                                | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.status`                                     | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.statusText`                                 | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.timeout`                                    | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.upload`                                     | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.withCredentials`                            | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.abort()`                                    | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.getAllResponseHeaders()`                    | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.getResponseHeader()`                        | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.open()`                                     | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.overrideMimeType()`                         | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.send()`                                     | `XMLHttpRequest` | `Async`      |
| `XMLHttpRequest.setRequestHeader()`                         | `XMLHttpRequest` | `Async`      |
| `fetch()`                                                   | `Fetch`          | `Async`      |
| `Headers`                                                   | `Fetch`          | `Async`      |
| `Headers.Headers()`                                         | `Fetch`          | `Async`      |
| `Headers.append()`                                          | `Fetch`          | `Async`      |
| `Headers.delete()`                                          | `Fetch`          | `Async`      |
| `Headers.entries()`                                         | `Fetch`          | `Async`      |
| `Headers.forEach()`                                         | `Fetch`          | `Async`      |
| `Headers.get()`                                             | `Fetch`          | `Async`      |
| `Headers.getSetCookie()`                                    | `Fetch`          | `Async`      |
| `Headers.has()`                                             | `Fetch`          | `Async`      |
| `Headers.keys()`                                            | `Fetch`          | `Async`      |
| `Headers.set()`                                             | `Fetch`          | `Async`      |
| `Headers.values()`                                          | `Fetch`          | `Async`      |
| `Request`                                                   | `Fetch`          | `Async`      |
| `Request.Request()`                                         | `Fetch`          | `Async`      |
| `Request.body`                                              | `Fetch`          | `Async`      |
| `Request.bodyUsed`                                          | `Fetch`          | `Async`      |
| `Request.cache`                                             | `Fetch`          | `Async`      |
| `Request.credentials`                                       | `Fetch`          | `Async`      |
| `Request.destination`                                       | `Fetch`          | `Async`      |
| `Request.headers`                                           | `Fetch`          | `Async`      |
| `Request.integrity`                                         | `Fetch`          | `Async`      |
| `Request.method`                                            | `Fetch`          | `Async`      |
| `Request.mode`                                              | `Fetch`          | `Async`      |
| `Request.redirect`                                          | `Fetch`          | `Async`      |
| `Request.referrer`                                          | `Fetch`          | `Async`      |
| `Request.referrerPolicy`                                    | `Fetch`          | `Async`      |
| `Request.signal Read only`                                  | `Fetch`          | `Async`      |
| `Request.url Read only`                                     | `Fetch`          | `Async`      |
| `Request.arrayBuffer()`                                     | `Fetch`          | `Async`      |
| `Request.blob()`                                            | `Fetch`          | `Async`      |
| `Request.clone()`                                           | `Fetch`          | `Async`      |
| `Request.formData()`                                        | `Fetch`          | `Async`      |
| `Request.json()`                                            | `Fetch`          | `Async`      |
| `Request.text()`                                            | `Fetch`          | `Async`      |
| `Response`                                                  | `Fetch`          | `Async`      |
| `Response.Response()`                                       | `Fetch`          | `Async`      |
| `Response.body`                                             | `Fetch`          | `Async`      |
| `Response.bodyUsed`                                         | `Fetch`          | `Async`      |
| `Response.headers`                                          | `Fetch`          | `Async`      |
| `Response.ok`                                               | `Fetch`          | `Async`      |
| `Response.redirected`                                       | `Fetch`          | `Async`      |
| `Response.status`                                           | `Fetch`          | `Async`      |
| `Response.statusText`                                       | `Fetch`          | `Async`      |
| `Response.type`                                             | `Fetch`          | `Async`      |
| `Response.url`                                              | `Fetch`          | `Async`      |
| `Response.error()`                                          | `Fetch`          | `Async`      |
| `Response.redirect()`                                       | `Fetch`          | `Async`      |
| `Response.arrayBuffer()`                                    | `Fetch`          | `Async`      |
| `Response.blob()`                                           | `Fetch`          | `Async`      |
| `Response.clone()`                                          | `Fetch`          | `Async`      |
| `Response.formData()`                                       | `Fetch`          | `Async`      |
| `Response.json()`                                           | `Fetch`          | `Async`      |
| `Response.text()`                                           | `Fetch`          | `Async`      |

[^async-function]: The `AsyncFunction` global object is not predefined in JavaScript, but rather defined as `const AsyncFunction = async function () {}.constructor;` (see: [`AsyncFunction`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AsyncFunction/AsyncFunction)).
Nonetheless, clients supporting the `Async` capability **MUST** provide the `AsyncFunction` global object defined as indicated above when executing a validator with the `Async` capability.
