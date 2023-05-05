<!-- markdownlint-disable MD043 -->

# VIP-01 --- JavaScript `"v-language"` Tag

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**,  **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

`v-language:javascript`

---

- [1. General Conventions](#1-general-conventions)
- [2. `NostrRead` Capability](#2-nostrread-capability)
- [3. `NostrValidate` Capability](#3-nostrvalidate-capability)
- [4. Available Built-In Objects](#4-available-built-in-objects)

---

## 1. General Conventions

The `<LANGUAGE>` placeholder **MUST** be `"javascript"`.

The validator definition event's `.content` **MUST** be ES6-compliant.

The `event` and `tagIndex` parameters are passed as pre-defined variables to the validator.

Return values are interpreted as JavaScript booleans.
Uncaught exceptions are considered `false` return values.

The execution environment will vary depending on the actual type of execution machine (ie. NodeJS vs browser-based), but the execution tactic should be compatible with extracting the value of the following expression:

```javascript
try {
  return Function(
    '"use strict";'
      + 'const event = arguments[0];'
      + 'const tagIndex = arguments[1];'
      + validatorEvent.content,
  ).apply(
    {},
    [
      event,
      tagIndex,
    ],
  );
} catch {
  return false;
}
```

where `event` and `tagIndex` are as above, and `validatorEvent` is the validator definition event referred to by ID in the event's `"v"` tag.

> If the client supports the `Async` capability (see [VIP-02](vip-02.md)) and the validator requests the `Async` capability, this expression becomes:
>
> ```javascript
> try {
>   return await AsyncFunction(
>     '"use strict";'
>       + 'const event = arguments[0];'
>       + 'const tagIndex = arguments[1];'
>       + validatorEvent.content,
>   ).apply(
>     {},
>     [
>       event,
>       tagIndex,
>     ],
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

## 4. Available Built-In Objects

The following table details what pre-defined objects are made available by default (note the absence of `globalThis` and `eval()`: these would enable the validator code to potentially break free from containment and are thus simply not provided at all; furthermore, note the absence of the default `Date` constructor and of `Date.now()`: these would make the validators non-pure):

| Symbol Name                                           |
| ----------------------------------------------------- |
| `Infinity`                                            |
| `NaN`                                                 |
| `undefined`                                           |
| `isFinite()`                                          |
| `isNaN()`                                             |
| `parseFloat()`                                        |
| `parseInt()`                                          |
| `decodeURI()`                                         |
| `decodeURIComponent()`                                |
| `encodeURI()`                                         |
| `encodeURIComponent()`                                |
| `Object`                                              |
| `Object.Object()`                                     |
| `Object.assign()`                                     |
| `Object.create()`                                     |
| `Object.defineProperties()`                           |
| `Object.defineProperty()`                             |
| `Object.entries()`                                    |
| `Object.freeze()`                                     |
| `Object.fromEntries()`                                |
| `Object.getOwnPropertyDescriptor()`                   |
| `Object.getOwnPropertyDescriptors()`                  |
| `Object.getOwnPropertyNames()`                        |
| `Object.getOwnPropertySymbols()`                      |
| `Object.getPrototypeOf()`                             |
| `Object.hasOwn()`                                     |
| `Object.is()`                                         |
| `Object.isExtensible()`                               |
| `Object.isFrozen()`                                   |
| `Object.isSealed()`                                   |
| `Object.keys()`                                       |
| `Object.preventExtensions()`                          |
| `Object.seal()`                                       |
| `Object.setPrototypeOf()`                             |
| `Object.values()`                                     |
| `Object.prototype.constructor`                        |
| `Object.prototype.hasOwnProperty()`                   |
| `Object.prototype.isPrototypeOf()`                    |
| `Object.prototype.propertyIsEnumerable()`             |
| `Object.prototype.toLocaleString()`                   |
| `Object.prototype.toString()`                         |
| `Object.prototype.valueOf()`                          |
| `Function`                                            |
| `Function.Function()`                                 |
| `Function.prototype.constructor`                      |
| `Function.length`                                     |
| `Function.name`                                       |
| `Function.prototype`                                  |
| `Function.prototype.apply()`                          |
| `Function.prototype.bind()`                           |
| `Function.prototype.call()`                           |
| `Function.prototype.toString()`                       |
| `Boolean`                                             |
| `Boolean.Boolean()`                                   |
| `Boolean.prototype.constructor`                       |
| `Boolean.prototype.toString()`                        |
| `Boolean.prototype.valueOf()`                         |
| `Symbol`                                              |
| `Symbol.Symbol()`                                     |
| `Symbol.hasInstance`                                  |
| `Symbol.isConcatSpreadable`                           |
| `Symbol.iterator`                                     |
| `Symbol.match`                                        |
| `Symbol.matchAll`                                     |
| `Symbol.replace`                                      |
| `Symbol.search`                                       |
| `Symbol.species`                                      |
| `Symbol.split`                                        |
| `Symbol.toPrimitive`                                  |
| `Symbol.toStringTag`                                  |
| `Symbol.unscopables`                                  |
| `Symbol.for()`                                        |
| `Symbol.keyFor()`                                     |
| `Symbol.prototype.constructor`                        |
| `Symbol.prototype.description`                        |
| `Symbol.prototype[@@toStringTag]`                     |
| `Symbol.prototype.toString()`                         |
| `Symbol.prototype.valueOf()`                          |
| `Symbol.prototype[@@toPrimitive]()`                   |
| `Error`                                               |
| `Error.Error()`                                       |
| `Error.prototype.constructor`                         |
| `Error.prototype.name`                                |
| `Error.cause`                                         |
| `Error.message`                                       |
| `Error.prototype.toString()`                          |
| `AggregateError`                                      |
| `AggregateError.AggregateError()`                     |
| `AggregateError.prototype.constructor`                |
| `AggregateError.prototype.name`                       |
| `AggregateError.errors`                               |
| `RangeError`                                          |
| `RangeError.RangeError()`                             |
| `RangeError.prototype.constructor`                    |
| `RangeError.prototype.name`                           |
| `ReferenceError`                                      |
| `ReferenceError.ReferenceError()`                     |
| `ReferenceError.prototype.constructor`                |
| `ReferenceError.prototype.name`                       |
| `TypeError`                                           |
| `TypeError.TypeError()`                               |
| `TypeError.prototype.constructor`                     |
| `TypeError.prototype.name`                            |
| `URIError`                                            |
| `URIError.URIError()`                                 |
| `URIError.prototype.constructor`                      |
| `URIError.prototype.name`                             |
| `Number`                                              |
| `Number.Number()`                                     |
| `Number.EPSILON`                                      |
| `Number.MAX_SAFE_INTEGER`                             |
| `Number.MAX_VALUE`                                    |
| `Number.MIN_SAFE_INTEGER`                             |
| `Number.MIN_VALUE`                                    |
| `Number.NaN`                                          |
| `Number.NEGATIVE_INFINITY`                            |
| `Number.POSITIVE_INFINITY`                            |
| `Number.isFinite()`                                   |
| `Number.isInteger()`                                  |
| `Number.isNaN()`                                      |
| `Number.isSafeInteger()`                              |
| `Number.parseFloat()`                                 |
| `Number.parseInt()`                                   |
| `Number.prototype.constructor`                        |
| `Number.prototype.toExponential()`                    |
| `Number.prototype.toFixed()`                          |
| `Number.prototype.toLocaleString()`                   |
| `Number.prototype.toPrecision()`                      |
| `Number.prototype.toString()`                         |
| `Number.prototype.valueOf()`                          |
| `BigInt`                                              |
| `BigInt.BigInt()`                                     |
| `BigInt.asIntN()`                                     |
| `BigInt.asUintN()`                                    |
| `BigInt.prototype.constructor`                        |
| `BigInt.prototype[@@toStringTag]`                     |
| `BigInt.prototype.toLocaleString()`                   |
| `BigInt.prototype.toString()`                         |
| `BigInt.prototype.valueOf()`                          |
| `Math`                                                |
| `Math[@@toStringTag]`                                 |
| `Math.E`                                              |
| `Math.LN2`                                            |
| `Math.LN10`                                           |
| `Math.LOG2E`                                          |
| `Math.LOG10E`                                         |
| `Math.PI`                                             |
| `Math.SQRT1_2`                                        |
| `Math.SQRT2`                                          |
| `Math.abs()`                                          |
| `Math.acos()`                                         |
| `Math.acosh()`                                        |
| `Math.asin()`                                         |
| `Math.asinh()`                                        |
| `Math.atan()`                                         |
| `Math.atanh()`                                        |
| `Math.atan2()`                                        |
| `Math.cbrt()`                                         |
| `Math.ceil()`                                         |
| `Math.clz32()`                                        |
| `Math.cos()`                                          |
| `Math.cosh()`                                         |
| `Math.exp()`                                          |
| `Math.expm1()`                                        |
| `Math.floor()`                                        |
| `Math.fround()`                                       |
| `Math.hypot()`                                        |
| `Math.imul()`                                         |
| `Math.log()`                                          |
| `Math.log1p()`                                        |
| `Math.log10()`                                        |
| `Math.log2()`                                         |
| `Math.max()`                                          |
| `Math.min()`                                          |
| `Math.pow()`                                          |
| `Math.random()`                                       |
| `Math.round()`                                        |
| `Math.sign()`                                         |
| `Math.sin()`                                          |
| `Math.sinh()`                                         |
| `Math.sqrt()`                                         |
| `Math.tan()`                                          |
| `Math.tanh()`                                         |
| `Math.trunc()`                                        |
| `Date`                                                |
| `Date.parse()`                                        |
| `Date.UTC()`                                          |
| `Date.prototype.getDate()`                            |
| `Date.prototype.getDay()`                             |
| `Date.prototype.getFullYear()`                        |
| `Date.prototype.getHours()`                           |
| `Date.prototype.getMilliseconds()`                    |
| `Date.prototype.getMinutes()`                         |
| `Date.prototype.getMonth()`                           |
| `Date.prototype.getSeconds()`                         |
| `Date.prototype.getTime()`                            |
| `Date.prototype.getTimezoneOffset()`                  |
| `Date.prototype.getUTCDate()`                         |
| `Date.prototype.getUTCDay()`                          |
| `Date.prototype.getUTCFullYear()`                     |
| `Date.prototype.getUTCHours()`                        |
| `Date.prototype.getUTCMilliseconds()`                 |
| `Date.prototype.getUTCMinutes()`                      |
| `Date.prototype.getUTCMonth()`                        |
| `Date.prototype.getUTCSeconds()`                      |
| `Date.prototype.setDate()`                            |
| `Date.prototype.setFullYear()`                        |
| `Date.prototype.setHours()`                           |
| `Date.prototype.setMilliseconds()`                    |
| `Date.prototype.setMinutes()`                         |
| `Date.prototype.setMonth()`                           |
| `Date.prototype.setSeconds()`                         |
| `Date.prototype.setTime()`                            |
| `Date.prototype.setUTCDate()`                         |
| `Date.prototype.setUTCFullYear()`                     |
| `Date.prototype.setUTCHours()`                        |
| `Date.prototype.setUTCMilliseconds()`                 |
| `Date.prototype.setUTCMinutes()`                      |
| `Date.prototype.setUTCMonth()`                        |
| `Date.prototype.setUTCSeconds()`                      |
| `Date.prototype.toDateString()`                       |
| `Date.prototype.toISOString()`                        |
| `Date.prototype.toJSON()`                             |
| `Date.prototype.toLocaleDateString()`                 |
| `Date.prototype.toLocaleString()`                     |
| `Date.prototype.toLocaleTimeString()`                 |
| `Date.prototype.toString()`                           |
| `Date.prototype.toTimeString()`                       |
| `Date.prototype.toUTCString()`                        |
| `Date.prototype.valueOf()`                            |
| `Date.prototype[@@toPrimitive]()`                     |
| `String`                                              |
| `String.String()`                                     |
| `String.fromCharCode()`                               |
| `String.fromCodePoint()`                              |
| `String.raw()`                                        |
| `String.prototype.constructor`                        |
| `String.length`                                       |
| `String.prototype.at()`                               |
| `String.prototype.charAt()`                           |
| `String.prototype.charCodeAt()`                       |
| `String.prototype.codePointAt()`                      |
| `String.prototype.concat()`                           |
| `String.prototype.endsWith()`                         |
| `String.prototype.includes()`                         |
| `String.prototype.indexOf()`                          |
| `String.prototype.isWellFormed()`                     |
| `String.prototype.lastIndexOf()`                      |
| `String.prototype.localeCompare()`                    |
| `String.prototype.match()`                            |
| `String.prototype.matchAll()`                         |
| `String.prototype.normalize()`                        |
| `String.prototype.padEnd()`                           |
| `String.prototype.padStart()`                         |
| `String.prototype.repeat()`                           |
| `String.prototype.replace()`                          |
| `String.prototype.replaceAll()`                       |
| `String.prototype.search()`                           |
| `String.prototype.slice()`                            |
| `String.prototype.split()`                            |
| `String.prototype.startsWith()`                       |
| `String.prototype.substring()`                        |
| `String.prototype.toLocaleLowerCase()`                |
| `String.prototype.toLocaleUpperCase()`                |
| `String.prototype.toLowerCase()`                      |
| `String.prototype.toString()`                         |
| `String.prototype.toUpperCase()`                      |
| `String.prototype.toWellFormed()`                     |
| `String.prototype.trim()`                             |
| `String.prototype.trimStart()`                        |
| `String.prototype.trimEnd()`                          |
| `String.prototype.valueOf()`                          |
| `String.prototype[@@iterator]()`                      |
| `RegExp`                                              |
| `RegExp.RegExp()`                                     |
| `RegExp[@@species]`                                   |
| `RegExp.prototype.constructor`                        |
| `RegExp.prototype.dotAll`                             |
| `RegExp.prototype.flags`                              |
| `RegExp.prototype.global`                             |
| `RegExp.prototype.hasIndices`                         |
| `RegExp.prototype.ignoreCase`                         |
| `RegExp.prototype.multiline`                          |
| `RegExp.prototype.source`                             |
| `RegExp.prototype.sticky`                             |
| `RegExp.prototype.unicode`                            |
| `RegExp.lastIndex`                                    |
| `RegExp.prototype.exec()`                             |
| `RegExp.prototype.test()`                             |
| `RegExp.prototype.toString()`                         |
| `RegExp.prototype[@@match]()`                         |
| `RegExp.prototype[@@matchAll]()`                      |
| `RegExp.prototype[@@replace]()`                       |
| `RegExp.prototype[@@search]()`                        |
| `RegExp.prototype[@@split]()`                         |
| `Array`                                               |
| `Array.Array()`                                       |
| `Array[@@species]`                                    |
| `Array.from()`                                        |
| `Array.isArray()`                                     |
| `Array.of()`                                          |
| `Array.prototype.constructor`                         |
| `Array.prototype[@@unscopables]`                      |
| `Array.length`                                        |
| `Array.prototype.at()`                                |
| `Array.prototype.concat()`                            |
| `Array.prototype.copyWithin()`                        |
| `Array.prototype.entries()`                           |
| `Array.prototype.every()`                             |
| `Array.prototype.fill()`                              |
| `Array.prototype.filter()`                            |
| `Array.prototype.find()`                              |
| `Array.prototype.findIndex()`                         |
| `Array.prototype.findLast()`                          |
| `Array.prototype.findLastIndex()`                     |
| `Array.prototype.flat()`                              |
| `Array.prototype.flatMap()`                           |
| `Array.prototype.forEach()`                           |
| `Array.prototype.group()`                             |
| `Array.prototype.groupToMap()`                        |
| `Array.prototype.includes()`                          |
| `Array.prototype.indexOf()`                           |
| `Array.prototype.join()`                              |
| `Array.prototype.keys()`                              |
| `Array.prototype.lastIndexOf()`                       |
| `Array.prototype.map()`                               |
| `Array.prototype.pop()`                               |
| `Array.prototype.push()`                              |
| `Array.prototype.reduce()`                            |
| `Array.prototype.reduceRight()`                       |
| `Array.prototype.reverse()`                           |
| `Array.prototype.shift()`                             |
| `Array.prototype.slice()`                             |
| `Array.prototype.some()`                              |
| `Array.prototype.sort()`                              |
| `Array.prototype.splice()`                            |
| `Array.prototype.toLocaleString()`                    |
| `Array.prototype.toReversed()`                        |
| `Array.prototype.toSorted()`                          |
| `Array.prototype.toSpliced()`                         |
| `Array.prototype.toString()`                          |
| `Array.prototype.unshift()`                           |
| `Array.prototype.values()`                            |
| `Array.prototype.with()`                              |
| `Array.prototype[@@iterator]()`                       |
| `TypedArray` _(Hidden)_                               |
| `TypedArray[@@species]`                               |
| `TypedArray.BYTES_PER_ELEMENT`                        |
| `TypedArray.from()`                                   |
| `TypedArray.of()`                                     |
| `TypedArray.prototype.buffer`                         |
| `TypedArray.prototype.byteLength`                     |
| `TypedArray.prototype.byteOffset`                     |
| `TypedArray.prototype.constructor`                    |
| `TypedArray.prototype.length`                         |
| `TypedArray.prototype[@@toStringTag]`                 |
| `TypedArray.prototype.BYTES_PER_ELEMENT`              |
| `TypedArray.prototype.at()`                           |
| `TypedArray.prototype.copyWithin()`                   |
| `TypedArray.prototype.entries()`                      |
| `TypedArray.prototype.every()`                        |
| `TypedArray.prototype.fill()`                         |
| `TypedArray.prototype.filter()`                       |
| `TypedArray.prototype.find()`                         |
| `TypedArray.prototype.findIndex()`                    |
| `TypedArray.prototype.findLast()`                     |
| `TypedArray.prototype.findLastIndex()`                |
| `TypedArray.prototype.forEach()`                      |
| `TypedArray.prototype.includes()`                     |
| `TypedArray.prototype.indexOf()`                      |
| `TypedArray.prototype.join()`                         |
| `TypedArray.prototype.keys()`                         |
| `TypedArray.prototype.lastIndexOf()`                  |
| `TypedArray.prototype.map()`                          |
| `TypedArray.prototype.reduce()`                       |
| `TypedArray.prototype.reduceRight()`                  |
| `TypedArray.prototype.reverse()`                      |
| `TypedArray.prototype.set()`                          |
| `TypedArray.prototype.slice()`                        |
| `TypedArray.prototype.some()`                         |
| `TypedArray.prototype.sort()`                         |
| `TypedArray.prototype.subarray()`                     |
| `TypedArray.prototype.toLocaleString()`               |
| `TypedArray.prototype.toReversed()`                   |
| `TypedArray.prototype.toSorted()`                     |
| `TypedArray.prototype.toString()`                     |
| `TypedArray.prototype.values()`                       |
| `TypedArray.prototype.with()`                         |
| `TypedArray.prototype[@@iterator]()`                  |
| `Int8Array`                                           |
| `Int8Array.Int8Array()`                               |
| `Int8Array.BYTES_PER_ELEMENT`                         |
| `Int8Array.prototype.BYTES_PER_ELEMENT`               |
| `Int8Array.prototype.constructor`                     |
| `Uint8Array`                                          |
| `Uint8Array.Uint8Array()`                             |
| `Uint8Array.BYTES_PER_ELEMENT`                        |
| `Uint8Array.prototype.BYTES_PER_ELEMENT`              |
| `Uint8Array.prototype.constructor`                    |
| `Uint8ClampedArray`                                   |
| `Uint8ClampedArray.Uint8ClampedArray()`               |
| `Uint8ClampedArray.BYTES_PER_ELEMENT`                 |
| `Uint8ClampedArray.prototype.BYTES_PER_ELEMENT`       |
| `Uint8ClampedArray.prototype.constructor`             |
| `Int16Array`                                          |
| `Int16Array.Int16Array()`                             |
| `Int16Array.BYTES_PER_ELEMENT`                        |
| `Int16Array.prototype.BYTES_PER_ELEMENT`              |
| `Int16Array.prototype.constructor`                    |
| `Uint16Array`                                         |
| `Uint16Array.Uint16Array()`                           |
| `Uint16Array.BYTES_PER_ELEMENT`                       |
| `Uint16Array.prototype.BYTES_PER_ELEMENT`             |
| `Uint16Array.prototype.constructor`                   |
| `Int32Array`                                          |
| `Int32Array.Int32Array()`                             |
| `Int32Array.BYTES_PER_ELEMENT`                        |
| `Int32Array.prototype.BYTES_PER_ELEMENT`              |
| `Int32Array.prototype.constructor`                    |
| `Uint32Array`                                         |
| `Uint32Array.Uint32Array()`                           |
| `Uint32Array.BYTES_PER_ELEMENT`                       |
| `Uint32Array.prototype.BYTES_PER_ELEMENT`             |
| `Uint32Array.prototype.constructor`                   |
| `BigInt64Array`                                       |
| `BigInt64Array.BigInt64Array()`                       |
| `BigInt64Array.BYTES_PER_ELEMENT`                     |
| `BigInt64Array.prototype.BYTES_PER_ELEMENT`           |
| `BigInt64Array.prototype.constructor`                 |
| `BigUint64Array`                                      |
| `BigUint64Array.BigUint64Array()`                     |
| `BigUint64Array.BYTES_PER_ELEMENT`                    |
| `BigUint64Array.prototype.BYTES_PER_ELEMENT`          |
| `BigUint64Array.prototype.constructor`                |
| `Float32Array`                                        |
| `Float32Array.Float32Array()`                         |
| `Float32Array.BYTES_PER_ELEMENT`                      |
| `Float32Array.prototype.BYTES_PER_ELEMENT`            |
| `Float32Array.prototype.constructor`                  |
| `Float64Array`                                        |
| `Float64Array.Float64Array()`                         |
| `Float64Array.BYTES_PER_ELEMENT`                      |
| `Float64Array.prototype.BYTES_PER_ELEMENT`            |
| `Float64Array.prototype.constructor`                  |
| `Map`                                                 |
| `Map.Map()`                                           |
| `Map[@@species]`                                      |
| `Map.prototype.constructor`                           |
| `Map.prototype.size`                                  |
| `Map.prototype[@@toStringTag]`                        |
| `Map.prototype.clear()`                               |
| `Map.prototype.delete()`                              |
| `Map.prototype.entries()`                             |
| `Map.prototype.forEach()`                             |
| `Map.prototype.get()`                                 |
| `Map.prototype.has()`                                 |
| `Map.prototype.keys()`                                |
| `Map.prototype.set()`                                 |
| `Map.prototype.values()`                              |
| `Map.prototype[@@iterator]()`                         |
| `Set`                                                 |
| `Set.Set()`                                           |
| `Set[@@species]`                                      |
| `Set.prototype.constructor`                           |
| `Set.prototype.size`                                  |
| `Set.prototype[@@toStringTag]`                        |
| `Set.prototype.add()`                                 |
| `Set.prototype.clear()`                               |
| `Set.prototype.delete()`                              |
| `Set.prototype.entries()`                             |
| `Set.prototype.forEach()`                             |
| `Set.prototype.has()`                                 |
| `Set.prototype.keys()`                                |
| `Set.prototype.values()`                              |
| `Set.prototype[@@iterator]()`                         |
| `WeakMap`                                             |
| `WeakMap.WeakMap()`                                   |
| `WeakMap.prototype.constructor`                       |
| `WeakMap.prototype[@@toStringTag]`                    |
| `WeakMap.prototype.delete()`                          |
| `WeakMap.prototype.get()`                             |
| `WeakMap.prototype.has()`                             |
| `WeakMap.prototype.set()`                             |
| `WeakSet`                                             |
| `WeakSet.WeakSet()`                                   |
| `WeakSet.prototype.constructor`                       |
| `WeakSet.prototype[@@toStringTag]`                    |
| `WeakSet.prototype.add()`                             |
| `WeakSet.prototype.delete()`                          |
| `WeakSet.prototype.has()`                             |
| `ArrayBuffer`                                         |
| `ArrayBuffer.ArrayBuffer()`                           |
| `ArrayBuffer[@@species]`                              |
| `ArrayBuffer.isView()`                                |
| `ArrayBuffer.prototype.byteLength`                    |
| `ArrayBuffer.prototype.constructor`                   |
| `ArrayBuffer.prototype.maxByteLength`                 |
| `ArrayBuffer.prototype.resizable`                     |
| `ArrayBuffer.prototype[@@toStringTag]`                |
| `ArrayBuffer.prototype.resize()`                      |
| `ArrayBuffer.prototype.slice()`                       |
| `DataView`                                            |
| `DataView.DataView()`                                 |
| `DataView.prototype.buffer`                           |
| `DataView.prototype.byteLength`                       |
| `DataView.prototype.byteOffset`                       |
| `DataView.prototype.constructor`                      |
| `DataView.prototype[@@toStringTag]`                   |
| `DataView.prototype.getBigInt64()`                    |
| `DataView.prototype.getBigUint64()`                   |
| `DataView.prototype.getFloat32()`                     |
| `DataView.prototype.getFloat64()`                     |
| `DataView.prototype.getInt16()`                       |
| `DataView.prototype.getInt32()`                       |
| `DataView.prototype.getInt8()`                        |
| `DataView.prototype.getUint16()`                      |
| `DataView.prototype.getUint32()`                      |
| `DataView.prototype.getUint8()`                       |
| `DataView.prototype.setBigInt64()`                    |
| `DataView.prototype.setBigUint64()`                   |
| `DataView.prototype.setFloat32()`                     |
| `DataView.prototype.setFloat64()`                     |
| `DataView.prototype.setInt16()`                       |
| `DataView.prototype.setInt32()`                       |
| `DataView.prototype.setInt8()`                        |
| `DataView.prototype.setUint16()`                      |
| `DataView.prototype.setUint32()`                      |
| `DataView.prototype.setUint8()`                       |
| `JSON`                                                |
| `JSON[@@toStringTag]`                                 |
| `JSON.parse()`                                        |
| `JSON.stringify()`                                    |
| `WeakRef`                                             |
| `WeakRef.WeakRef()`                                   |
| `WeakRef.prototype[@@toStringTag]`                    |
| `Iterator`                                            |
| `Iterator.prototype[@@iterator]()`                    |
| `GeneratorFunction`                                   |
| `GeneratorFunction.GeneratorFunction()`               |
| `GeneratorFunction.prototype.constructor`             |
| `GeneratorFunction.prototype.prototype`               |
| `GeneratorFunction.prototype[@@toStringTag]`          |
| `Generator`                                           |
| `Generator.prototype.constructor`                     |
| `Generator.prototype[@@toStringTag]`                  |
| `Generator.prototype.next()`                          |
| `Generator.prototype.return()`                        |
| `Generator.prototype.throw()`                         |
| `Intl`                                                |
| `Intl.Collator()`                                     |
| `Intl.DateTimeFormat()`                               |
| `Intl.DisplayNames()`                                 |
| `Intl.DurationFormat()`                               |
| `Intl.ListFormat()`                                   |
| `Intl.Locale()`                                       |
| `Intl.NumberFormat()`                                 |
| `Intl.PluralRules()`                                  |
| `Intl.RelativeTimeFormat()`                           |
| `Intl.Segmenter()`                                    |
| `Intl.getCanonicalLocales()`                          |
| `Intl.supportedValuesOf()`                            |
| `Intl.Collator`                                       |
| `Intl.Collator.Collator()`                            |
| `Intl.Collator.supportedLocalesOf()`                  |
| `Intl.Collator.prototype.constructor`                 |
| `Intl.Collator.prototype[@@toStringTag]`              |
| `Intl.Collator.prototype.compare()`                   |
| `Intl.Collator.prototype.resolvedOptions()`           |
| `Intl.DateTimeFormat`                                 |
| `Intl.DateTimeFormat.DateTimeFormat()`                |
| `Intl.DateTimeFormat.supportedLocalesOf()`            |
| `Intl.DateTimeFormat.prototype.constructor`           |
| `Intl.DateTimeFormat.prototype[@@toStringTag]`        |
| `Intl.DateTimeFormat.prototype.format()`              |
| `Intl.DateTimeFormat.prototype.formatRange()`         |
| `Intl.DateTimeFormat.prototype.formatRangeToParts()`  |
| `Intl.DateTimeFormat.prototype.formatToParts()`       |
| `Intl.DateTimeFormat.prototype.resolvedOptions()`     |
| `Intl.DisplayNames`                                   |
| `Intl.DisplayNames.DisplayNames()`                    |
| `Intl.DisplayNames.supportedLocalesOf()`              |
| `Intl.DisplayNames.prototype.constructor`             |
| `Intl.DisplayNames.prototype[@@toStringTag]`          |
| `Intl.DisplayNames.prototype.of()`                    |
| `Intl.DisplayNames.prototype.resolvedOptions()`       |
| `Intl.ListFormat`                                     |
| `Intl.ListFormat.ListFormat()`                        |
| `Intl.ListFormat.supportedLocalesOf()`                |
| `Intl.ListFormat.prototype.constructor`               |
| `Intl.ListFormat.prototype[@@toStringTag]`            |
| `Intl.ListFormat.prototype.format()`                  |
| `Intl.ListFormat.prototype.formatToParts()`           |
| `Intl.ListFormat.prototype.resolvedOptions()`         |
| `Intl.Locale`                                         |
| `Intl.Locale.Locale()`                                |
| `Intl.Locale.prototype.baseName`                      |
| `Intl.Locale.prototype.calendar`                      |
| `Intl.Locale.prototype.calendars`                     |
| `Intl.Locale.prototype.caseFirst`                     |
| `Intl.Locale.prototype.collation`                     |
| `Intl.Locale.prototype.collations`                    |
| `Intl.Locale.prototype.constructor`                   |
| `Intl.Locale.prototype.hourCycle`                     |
| `Intl.Locale.prototype.hourCycles`                    |
| `Intl.Locale.prototype.language`                      |
| `Intl.Locale.prototype.numberingSystem`               |
| `Intl.Locale.prototype.numberingSystems`              |
| `Intl.Locale.prototype.numeric`                       |
| `Intl.Locale.prototype.region`                        |
| `Intl.Locale.prototype.script`                        |
| `Intl.Locale.prototype.textInfo`                      |
| `Intl.Locale.prototype.timeZones`                     |
| `Intl.Locale.prototype.weekInfo`                      |
| `Intl.Locale.prototype[@@toStringTag]`                |
| `Intl.Locale.prototype.maximize()`                    |
| `Intl.Locale.prototype.minimize()`                    |
| `Intl.Locale.prototype.toString()`                    |
| `Intl.NumberFormat`                                   |
| `Intl.NumberFormat.NumberFormat()`                    |
| `Intl.NumberFormat.supportedLocalesOf()`              |
| `Intl.NumberFormat.prototype.constructor`             |
| `Intl.NumberFormat.prototype[@@toStringTag]`          |
| `Intl.NumberFormat.prototype.format()`                |
| `Intl.NumberFormat.prototype.formatRange()`           |
| `Intl.NumberFormat.prototype.formatRangeToParts()`    |
| `Intl.NumberFormat.prototype.formatToParts()`         |
| `Intl.NumberFormat.prototype.resolvedOptions()`       |
| `Intl.PluralRules`                                    |
| `Intl.PluralRules.PluralRules()`                      |
| `Intl.PluralRules.supportedLocalesOf()`               |
| `Intl.PluralRules.prototype.constructor`              |
| `Intl.PluralRules.prototype[@@toStringTag]`           |
| `Intl.PluralRules.prototype.resolvedOptions()`        |
| `Intl.PluralRules.prototype.select()`                 |
| `Intl.PluralRules.prototype.selectRange()`            |
| `Intl.RelativeTimeFormat`                             |
| `Intl.RelativeTimeFormat.RelativeTimeFormat()`        |
| `Intl.RelativeTimeFormat.supportedLocalesOf()`        |
| `Intl.RelativeTimeFormat.prototype.constructor`       |
| `Intl.RelativeTimeFormat.prototype[@@toStringTag]`    |
| `Intl.RelativeTimeFormat.prototype.format()`          |
| `Intl.RelativeTimeFormat.prototype.formatToParts()`   |
| `Intl.RelativeTimeFormat.prototype.resolvedOptions()` |
| `Intl.Segmenter`                                      |
| `Intl.Segmenter.Segmenter()`                          |
| `Intl.Segmenter.supportedLocalesOf()`                 |
| `Intl.Segmenter.prototype.constructor`                |
| `Intl.Segmenter.prototype[@@toStringTag]`             |
| `Intl.Segmenter.prototype.resolvedOptions()`          |
| `Intl.Segmenter.prototype.segment()`                  |
