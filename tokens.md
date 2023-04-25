<!-- markdownlint-enable -->
<!-- markdownlint-disable MD013 -->

# NIP-XX --- NOSTR Tokens

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**,  **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

`kind:1001`
`tag:y` `tag:z`

`depends:01` `depends:12` `depends:16`

---

- [1. Motivation](#1-motivation)
- [2. Short Description](#2-short-description)
- [3. Overview](#3-overview)
- [4. The NOSTR Token Flow Event](#4-the-nostr-token-flow-event)
- [5. Client Behavior](#5-client-behavior)
  - [5.1. Additional Restrictions](#51-additional-restrictions)
  - [5.2. Token Ownership Checks](#52-token-ownership-checks)
  - [5.3. Validators](#53-validators)
    - [5.3.1. Token Flow](#531-token-flow)
    - [5.3.2. Token Ownership Check](#532-token-ownership-check)
- [6. FAQ](#6-faq)

---

## 1. Motivation

## 2. Short Description

## 3. Overview

## 4. The NOSTR Token Flow Event

## 5. Client Behavior

### 5.1. Additional Restrictions

### 5.2. Token Ownership Checks

### 5.3. Validators

#### 5.3.1. Token Flow

#### 5.3.2. Token Ownership Check

## 6. FAQ

---

```json
{
    ...,
    "kind": 1001,
    ...,
    "pubkey": <PUBKEY>,
    ...,
    "tags": [
        ...,
        ["y", <TOKEN_ID>],  // Used input IDs
        ...,
        ["z", <TOKEN_ID>],  // Used output IDs
        ...
        ["p", <DESTINATION>],  // Mentioned destination pubkeys --- NOTE: the recommended relay URL, if given, is ignored
        ...
    ],
    ...,
    "content": <CONTENT>,  // the JSON string serialization of the content object detailed below
    ...
}
```

```json
{
    "inputs": [
        ...,
        {
            "id" <TOKEN_ID>,  // UUIDv4 --- the ID of an unburned output
            "nonce": <NONCE>  // NONCE --- the NONCE of said output, such that SHA-256(NONCE) == Output(ID).commitment
        }
        ...
    ],
    "outputs": [
        ...,
        {
            "id": <TOKEN_ID>,              // UUIDv4 --- a random ID to associate to this output
            "commitment": <COMMITMENT>,    // SHA-256 of NONCE --- public commitment to the value of NONCE
            "secret": <SECRET>,            // Encrypt(DESTINATION, NONCE) --- private revelation of the value of NONCE
            "destination": <DESTINATION>,  // PubKey --- the PubKey of the output's destination,
            "quantity": <QUANTITY>         // INT --- a number representing the amount sent of the Token in question
        },
        ...
    ]
}
```

All input IDs mentioned in `.inputs.*.id` must appear in `"y"` tags.

All output IDs mentioned in `.outputs.*.id` must appear in `"z"` tags.

## Checks

1. All `.inputs.*.id` values should correspond to existing `.outputs.*.id` values, for which there's no other `kind:1001` message mentioning them in their own `.inputs.*.id` values.
2. All such `.outputs.*.id` values should have `PUBKEY` as their corresponding `.outputs.*.destination` values.
3. For each `.inputs.*.id`, check that applying `SHA-256` to the corresponding `.inputs.*.nonce` value equals the corresponding `.outputs.*.commitment` value.

In pseudocode:

1. For each `input` in `.inputs`:
    1. Verify that `NostrRead([{"kind": 1001, "#y": "${input.id}"}])` returns no results.
    2. Verify that `NostrRead([{"kind": 1001, "#z": "${input.id}"}])` returns a single result.
    3. Let `output` be the single value returned therein.
    4. Verify that `output.destination` equals `.pubkey`.
    5. Verify that `SHA-256(input.nonce)` equals `output.commitment`.

As a [validator](validators.md):

```javascript
// Requires the "NostrRead", "NostrValidate", "Crypto", and "Async" capabilities

/**
 * NOSTR Tokens Validator
 *
 * This generic NOSTR Tokens Validator can be attached to a NOSTR Token Flow Event to validate
 * the actions taken therein.
 * It has been coded so as to allow several different policies, by tweaking the values of:
 *
 * - BURN: a boolean, determining whether it is permitted to burn tokens away
 * - RELAYS: a string Set with the URLs of the preferred relays to use; consensus is required of at
 *     least MORE than half of them
 * - PRE_MINTED: a Set of outputs that are considered pre-minted and pre-assigned
 * - ALLOWED_MINTERS: a Set of pubkeys or prefixes thereof that are allowed to mint tokens
 *
 * Setting values for ALLOWED_MINTERS allows for the emission policy to be controlled tightly, only
 * allowing some, all, or none to add liquidity.
 * For extreme examples, consider:
 *
 * - ALLOWED_MINTERS = new Set(): this allows no-one to mint tokens, all tokens in existence are those
 *     provided in the PRE_MINTED array and none more (consequently, if the PRE_MINTED array is
 *     empty, these tokens basically become impossible to use at all)
 * - ALLOWED_MINTERS = new Set([""]): this allows everyone to mint tokens, since "" is a prefix of any
 *     pubkey, anyone can create liquidity
 *
 * Setting a value for PRE_MINTED makes it so that the given outputs are originally available, thus
 * it allows for emission outside of the dynamic control of ALLOWED_MINTERS.
 * This makes it possible to implement, eg, finite-supply tokens (setting ALLOWED_MINTERS to Set()).
 *
 * Usage merely requires adding a tag of the form:
 *
 *     ["v", "<VALIDATOR_MESSAGE_ID>"]
 *
 * to NOSTR Token Flow events.
 *
 */

const BURN = true;                  // whether burning Tokens is allowed
const RELAYS = new Set();           // a set of relay URLs to use
const PRE_MINTED = new Set();       // a set of pre-minted outputs
const ALLOWED_MINTERS = new Set();  // a set of pubkeys or prefixes that are indeed allowed to mint

/**
 * Check if two sets contain the same elements
 *
 * @param {Set} a - First set to compare
 * @param {Set} b - Second set to compare
 * @return {boolean}  True if both sets contain the same elements, false otherwise
 */
function equalSets(a, b) {
    return a.size === b.size && new Set([...a, ...b]).size === a.size;
}

/**
 * Calculate the SHA-256 of the given string, and return it as a hex string
 *
 * @param {string} data - The data to hash
 * @return {string}  The resulting hex string
 */
async function sha256toHex(data) {
    return Array.from(
            new Uint8Array(
                await crypto.subtle.digest(
                    "SHA-256",
                    (new TextEncoder()).encode(data),
                )
            )
        )
        .map(bytes => bytes.toString(16).padStart(2, "0"))
        .join("")
    ;
}

/**
 * Retrieve the valid NOSTR events selected by the given filter from the given relay
 *
 * @param {object[]} filters - Filter to apply
 * @param {string} relay - Relay to use
 * @return {object[]}  The resulting events
 */
function nostrReadValidated(filters, relay) {
    return NOSTR.read(filter, relay).filter(NOSTR.validate);
}

/**
 * Retrieve the NOSTR Token Flow events using the given input or output IDs, up until the given moment
 *
 * @param {"y"|"z"} tagName - the tag to search for
 * @param {number[]} ids - The input IDs to search for
 * @param {number} until - The maximum timestamp to consider
 * @return {object[]}  The resulting events
 */
function fetchFromRelays(tagName, ids, until) {
    const filter = [{"kinds": [1001], `#${tagName}`: ids, "until": until}];

    var entries = {};
    for (const RELAY of RELAYS) {
        for (const event of nostrReadValidated(filter, RELAY)) {
            if (!entries.has(event.id)) {
                entries[event.id] = {"count": 0, "event": event};
            }
            entries[event.id].count++;
        }
    }

    return entries
        .values()
        .filter(entry => (RELAYS.length >> 1) < entry.count)
        .map(entry => entry.event)
    ;
}

/**
 * Retrieve the input blocks using the given input IDs, up until the given moment
 *
 * @param {number[]} ids - The input IDs to search for
 * @param {number} until - The maximum timestamp to consider
 * @return {object[]}  The input blocks using the given input IDs
 */
function fetchInputs(ids, until) {
    return fetchFromRelays("y", ids, until)
        .flatMap(event => JSON.parse(event.content).inputs)
        .filter(input => ids.has(input.id))
    ;
}

/**
 * Retrieve the output blocks using the given output IDs, up until the given moment
 *
 * @param {number[]} ids - The output IDs to search for
 * @param {number} until - The maximum timestamp to consider
 * @return {object[]}  The output blocks using the given output IDs
 */
function fetchOutputs(ids, until) {
    return fetchFromRelays("z", ids, until)
        .flatMap(event => JSON.parse(event.content).outputs)
        .concat(Array.from(PRE_MINTED))
        .filter(output => ids.has(output.id))
    ;
}

/**
 * Retrieve the set of first-values associated to the tag name given in the event given
 *
 * @param {string} tagName - The tag name to look for
 * @param {object} event - The event to extract tag values from
 * @return {Set}  The set of values
 */
function tagValues(tagName, event) {
    var result = new Set(
        events.tags
            .map(tag => tag[0] === tagName ? tag[1] : undefined)
    );
    result.delete(undefined);
    return result;
}


const { event, tagIndex } = JSON.parse(arguments[0]);  // decode the input argument, extract event and tagIndex

const [ tagName ] = event.tags[tagIndex];  // extract the validator tag name

if (tagName !== "v") {      // (OPTIONAL) verify that we are indeed passed a validator tag
    return false;           // fail if we're not
}                           //
if (event.kind !== 1001) {  // verify that we're being run on a Token Flow event
    return false;           // fail if we're not
}                           //

const taggedInputs = tagValues("y", event);        // retrieve the set of all input tags
const taggedOutputs = tagValues("z", event);       // retrieve the set of all output tags
const taggedDestinations = tagValues("p", event);  // retrieve the set of all destination tags

var seenInputs = new Set();        // initialize seen inputs
var seenOutputs = new Set();       // initialize seen outputs
var seenDestinations = new Set();  // initialize seen destinations

const content = JSON.parse(event.content);  // parse the event's content

var total = 0n;  // keep running total of how many funds are moved (use BigInt)

for (const input of content.inputs) {                            // iterate through each input
    if (seenInputs.has(input.id)) {                              // verify the input ID is not repeated
        return false;                                            // fail if it is
    }                                                            //
    if ([] !== fetchInputs([input.id], event.created_at)) {      // verify that the input is not
        return false;                                            // already burnt, fail otherwise
    }                                                            //
    const outputs = fetchOutputs([input.id], event.created_at);  // retrieve all outputs associated to this input
    if (1 !== outputs.length) {                                  // verify there's only one of them
        return false;                                            // fail otherwise
    }                                                            //
    if (output[0].destination != event.pubkey) {                 // verify it's directed to us,
        return false;                                            // fail otherwise
    }                                                            //
    if (sha256toHex(input.nonce) != output[0].commitment) {      // verify it matches the commitment,
        return false;                                            // fail otherwise
    }                                                            //
    total += output[0].quantity;                                 // accumulate running total
    seenInputs.add(input.id);                                    // add the current input ID to the seen ones
}                                                                //

for (const output of content.outputs) {        // iterate through each output
    if (seenOutputs.has(output.id)) {          // verify the output ID is not repeated
        return false;                          // fail if it is
    }                                          //
    total -= output.quantity;                  // decrease running total
    seenOutputs.add(output.id);                // add the current output ID to the seen ones
    seenDestinations.add(output.destination);  // add the destination ID to the seen ones
}                                              //

if (fetchOutputs(Array.from(seenOutputs), event.created_at) !== []) {  // make sure the newly-created outputs are new
    return false;                                                      // fail otherwise
}                                                                      //

if (!equalSets(taggedInputs, seenInputs)) {              // check that all seen inputs are tagged
    return false;                                        // fail if not
}                                                        //
if (!equalSets(taggedOutputs, seenOutputs)) {            // check that all seen outputs are tagged
    return false;                                        // fail if not
}                                                        //
if (!equalSets(taggedDestinations, seenDestinations)) {  // check that all seen destinations are tagged
    return false;                                        // fail if not
}                                                        //

if (total < 0) {                                                        // if we're minting tokens
    return Array.from(ALLOWED_MINTERS)                                  // check that at least one allowed minter
        .some(allowedMinter => event.pubkey.startsWith(allowedMinter))  // is a prefix of the current pubkey
    ;
} else if (0 < total) {                                // if we're burning tokens
    return BURN;                                       // this is valid only if burning allowed
} else {                                               // otherwise
    return true;                                       // everything looks fine, validate
}                                                      //
```

---

```javascript
// Requires the "NostrRead", "NostrValidate", and "Async" capabilities

/**
 * NOSTR Token Ownership
 *
 * This generic NOSTR Token Ownership validator can be attached to a NOSTR Event to validate that the
 * event sender indeed has ownership of the given amount of NOSTR Tokens at the given moment.
 * It has been coded so as to interoperate with the NOSTR Tokens Validator above:
 *
 * - RELAYS: a string Set with the URLs of the preferred relays to use; consensus is required of at
 *     least MORE than half of them
 * - PRE_MINTED: a Set of outputs that are considered pre-minted and pre-assigned
 *
 * Setting a value for PRE_MINTED makes it so that the given outputs are originally available, it is
 * required to be in sync with the on ein the corresponding NOSTR Tokens Validator.
 *
 * Usage merely requires adding a tag of the form:
 *
 *     ["v", "<VALIDATOR_MESSAGE_ID>", "<TARGET_AMOUNT>", "<TOKEN_ID_1>", "<TOKEN_ID_2>", ..., "<TOKEN_ID_N>", ...]
 *
 * to NOSTR events.
 *
 */

const RELAYS = new Set();      // a set of relay URLs to use
const PRE_MINTED = new Set();  // a set of pre-minted outputs

/**
 * Retrieve the valid NOSTR events selected by the given filter from the given relay
 *
 * @param {object[]} filters - Filter to apply
 * @param {string} relay - Relay to use
 * @return {object[]}  The resulting events
 */
function nostrReadValidated(filters, relay) {
    return NOSTR.read(filter, relay).filter(NOSTR.validate);
}

/**
 * Retrieve the NOSTR Token Flow events using the given input or output IDs, up until the given moment
 *
 * @param {"y"|"z"} tagName - the tag to search for
 * @param {number[]} ids - The input IDs to search for
 * @param {number} until - The maximum timestamp to consider
 * @return {object[]}  The resulting events
 */
function fetchFromRelays(tagName, ids, until) {
    const filter = [{"kinds": [1001], `#${tagName}`: ids, "until": until}];

    var entries = {};
    for (const RELAY of RELAYS) {
        for (const event of nostrReadValidated(filter, RELAY)) {
            if (!entries.has(event.id)) {
                entries[event.id] = {"count": 0, "event": event};
            }
            entries[event.id].count++;
        }
    }

    return entries
        .values()
        .filter(entry => (RELAYS.length >> 1) < entry.count)
        .map(entry => entry.event)
    ;
}

/**
 * Retrieve the input blocks using the given input IDs, up until the given moment
 *
 * @param {number[]} ids - The input IDs to search for
 * @param {number} until - The maximum timestamp to consider
 * @return {object[]}  The input blocks using the given input IDs
 */
function fetchInputs(ids, until) {
    return fetchFromRelays("y", ids, until)
        .flatMap(event => JSON.parse(event.content).inputs)
        .filter(input => ids.has(input.id))
    ;
}

/**
 * Retrieve the output blocks using the given output IDs, up until the given moment
 *
 * @param {number[]} ids - The output IDs to search for
 * @param {number} until - The maximum timestamp to consider
 * @return {object[]}  The output blocks using the given output IDs
 */
function fetchOutputs(ids, until) {
    return fetchFromRelays("z", ids, until)
        .flatMap(event => JSON.parse(event.content).outputs)
        .concat(Array.from(PRE_MINTED))
        .filter(output => ids.has(output.id))
    ;
}


const { event, tagIndex } = JSON.parse(arguments[0]);  // decode the input argument, extract event and tagIndex

const [ tagName, , targetAmount, ...tokenIds ] = event.tags[tagIndex];  // extract the validator tag name and output IDs

if (tagName !== "v") {  // (OPTIONAL) verify that we are indeed passed a validator tag
    return false;       // fail if we're not
}                       //

var target = BigInt(targetAmount);

const tokenIdsClean = Set(tokenIds);
const outputs = fetchOutputs(Array.from(tokenIdsClean), event.created_at)
    .filter(output => output.destination === event.pubkey)
;

if (outputs.length !== tokenIdsClean.length) {
    return false;
}
if (fetchInputs(Array.from(tokenIdsClean), event.created_at) !== []) {
    return false;
}

for (const output of outputs) {
    target -= output.quantity;
}

return target <= 0;
```

### Additional checks

1. If there are more outputs than inputs, new tokens are being minted.
2. If there are more inputs than outputs, then tokens are being burned.

Implementations can enforce further checks using these facts.

## Extensions

An output object may include a `.quantity` field, specifying how many fungible units of the token being minted this output is good for.
When used as input though, ALL such units are burned, and the user generating the event is responsible for sending the unused token units to themselves.

Needless to say, the checks need to take this new field into account when validating.
