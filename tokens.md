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
  - [4.1. Validator](#41-validator)
- [5. Client Behavior](#5-client-behavior)
- [6. FAQ](#6-faq)

---

## 1. Motivation

## 2. Short Description

## 3. Overview

## 4. The NOSTR Token Flow Event

### 4.1. Validator

## 5. Client Behavior

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
        ["y", <INPUT_ID>],
        ...,
        ["z", <OUTPUT_ID>],
        ...
    ],
    ...,
    "content": {
        "inputs": [
            ...,
            {
                "id" <ID>,        // UUIDv4 --- the ID of an unburned output
                "nonce": <NONCE>  // NONCE --- the NONCE of said output, such that SHA-256(NONCE) == Output(ID).commitment
            }
            ...
        ],
        "outputs": [
            ...,
            {
                "id": <ID>,                    // UUIDv4 --- a random ID to associate to this output
                "commitment": <COMMITMENT>,    // SHA-256 of NONCE --- public commitment to the value of NONCE
                "secret": <SECRET>,            // Encrypt(DESTINATION, NONCE) --- private revelation of the value of NONCE
                "destination": <DESTINATION>,  // PubKey --- the PubKey of the output's destination,
                "quantity": <QUANTITY>
            },
            ...
        ]
    },
    ...
}
```

All input IDs mentioned in `.content.inputs.*.id` must appear in `"y"` tags.

All output IDs mentioned in `.content.outputs.*.id` must appear in `"z"` tags.

## Checks

1. All `.content.inputs.*.id` values should correspond to existing `.content.outputs.*.id` values, for which there's no other `kind:1001` message mentioning them in their own `.content.inputs.*.id` values.
2. All such `.content.outputs.*.id` values should have `PUBKEY` as their corresponding `.content.outputs.*.destination` values.
3. For each `.content.inputs.*.id`, check that applying `SHA-256` to the corresponding `.content.inputs.*.nonce` value equals the corresponding `.content.outputs.*.commitment` value.

In pseudocode:

1. For each `input` in `.content.inputs`:
    1. Verify that `NostrRead([{"kind": 1001, "#y": "${input.id}"}])` returns no results.
    2. Verify that `NostrRead([{"kind": 1001, "#z": "${input.id}"}])` returns a single result.
    3. Let `output` be the single value returned therein.
    4. Verify that `output.destination` equals `.pubkey`.
    5. Verify that `SHA-256(input.nonce)` equals `output.commitment`.

As a [validator](validators.md):

```javascript
// Requires the "NostrRead", "Crypto", and "Async" capabilities

/**
 * NOSTR Tokens Validator
 *
 * This generic NOSTR Tokens Validator can be attached to a NOSTR Token Flow Event to validate
 * the actions taken therein.
 * It has been coded so as to allow several different policies, by tweaking the values of:
 *
 * - RELAY: a sting with the URL of the preferred relay to use; it is HIGHLY RECOMMENDED to set this
 *     value to the relay that will act as the single source of truth for al these NOSTR Tokens
 * - PRE_MINTED: an array of outputs that are considered pre-minted and pre-assigned
 * - ALLOWED_MINTERS: an array of pubkeys or prefixes thereof that are allowed to mint tokens
 *
 * Setting a value for RELAY is HIGHLY RECOMMENDED because otherwise, thanks to the distributed nature
 * of the NOSTR network, double-spend attacks are easy to realize, having a single relay act as the
 * arbiter of truth reduces this possibility considerably.
 *
 * Setting values for ALLOWED_MINTERS allows for the emission policy to be controlled tightly, only
 * allowing some, all, or none to add liquidity.
 * For extreme examples, consider:
 *
 * - ALLOWED_MINTERS = []: this allows no-one to mint tokens, all tokens in existence are those
 *     provided in the PRE_MINTED array and none more (consequently, if the PRE_MINTED array is
 *     empty, these tokens basically become impossible to use at all)
 * - ALLOWED_MINTERS = ['']: this allows everyone to mint tokens, since '' is a prefix of any
 *     pubkey, anyone can create liquidity
 *
 * Setting a value for PRE_MINTED makes it so that the given outputs are originally available, thus
 * it allows for emission outside of the dynamic control of ALLOWED_MINTERS.
 * This makes it possible to implement, eg, finite-supply tokens (setting ALLOWED_MINTERS to []).
 *
 * Usage merely requires adding a tag of the form:
 *
 *     ["v", "<VALIDATOR_MESSAGE_ID>"]
 *
 * to NOSTR Token Flow events.
 *
 */

const RELAY = "";            // the URL of the conflict resolution relay to use
const PRE_MINTED = [];       // an array of pre-minted outputs
const ALLOWED_MINTERS = [];  // an array of pubkeys or prefixes that are indeed allowed to mint

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
    .map((bytes) => bytes.toString(16).padStart(2, "0"))
    .join("")
  ;
}

/**
 * Retrieve the NOSTR Token Flow events using the given input ID
 *
 * @param {number} id - The input ID to search for
 * @return {object[]}  The events using the given input ID
 */
function fetchInputs(id) {
    return RELAY !== ""
        ? NOSTR.read([{"kind": 1001, "#y": id}], RELAY)
        : NOSTR.read([{"kind": 1001, "#y": id}])
    ;
}

/**
 * Retrieve the NOSTR Token Flow events using the given output ID
 *
 * @param {number} id - The output ID to search for
 * @return {object[]}  The events using the given output ID
 */
function fetchOutputs(id) {
    return RELAY !== ""
        ? NOSTR.read([{"kind": 1001, "#z": id}], RELAY)
        : NOSTR.read([{"kind": 1001, "#z": id}])
    ;
}

const { event, tagIndex } = JSON.parse(arguments[0]);  // decode the input argument, extract event and tagIndex

const [ tagName ] = event.tags[tagIndex];  // extract the validator tag name

if (tagName !== "v") {      // (OPTIONAL) verify that we are indeed passed a validator tag
  return false;             // fail if we're not
}                           //
if (event.kind !== 1001) {  // verify that we're being run on a Token Flow event
    return false;           // fail of re're not
}                           //

var total = 0n;  // keep running total of how many funds are created (use BigInt)

for (const input of event.content.inputs) {               // iterate through each input
    if (fetchInputs(input.id) != []) {                    // verify that the input is not
        return false;                                     // already burnt, fail otherwise
    }                                                     //
    var outputs = fetchOutputs(input.id);                 // retrieve all outputs associated to this input
    if (1 < outputs.length) {                             // check there's at most one,
        return false;                                     // fail otherwise
    } else if (outputs.length < 1) {                      // if we couldn't find outputs...
        for (const preMinted of PRE_MINTED) {             // ... check pre-minted
            if (preMinted.id === input.id) {              // if the pre-minted output has the same ID
                outputs.push(preMinted);                  // accumulate it
            }                                             //
        }                                                 //
        if (1 !== outputs.length) {                       // check that there's exactly one pre-minted output
            return false;                                 // fail otherwise
        }                                                 //
    }                                                     //
    const [ output ] = outputs;                           // keep the output
    if (output.destination != event.pubkey) {             // verify it's directed to us,
        return false;                                     // fail otherwise
    }                                                     //
    if (sha256toHex(input.nonce) != output.commitment) {  // verify it matches the commitment,
        return false;                                     // fail otherwise
    }                                                     //
    total += output.quantity;                             // accumulate running total
}                                                         //

for (const output of event.content.outputs) {  // iterate through each output
    total -= output.quantity;                  // decrease running total
}                                              //

if (total <= 0) {                                      // if we're minting tokens
    for (const allowedMinter of ALLOWED_MINTERS) {     // iterate through each allowed minter
        if (event.pubkey.startsWith(allowedMinter)) {  // if the event's pubkey os a prefix or equal
            return true;                               // this is a valid event
        }                                              //
    }                                                  //
    return false;                                      // if not an allowed minter, fail
}

return true;  // everything looks fine, validate
```

### Additional checks

1. If there are more outputs than inputs, new tokens are being minted.
2. If there are more inputs than outputs, then tokens are being burned.

Implementations can enforce further checks using these facts.

## Extensions

An output object may include a `.quantity` field, specifying how many fungible units of the token being minted this output is good for.
When used as input though, ALL such units are burned, and the user generating the event is responsible for sending the unused token units to themselves.

Needless to say, the checks need to take this new field into account when validating.
