<!-- markdownlint-enable -->
<!-- markdownlint-disable MD013 -->

# NIP-XX --- NOSTR Tokens

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**,  **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

`kind:1001`
`tag:y` `tag:z`

`depends:01` `depends:12` `depends:16`

---

---

## Token Flow Event

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

const ALLOWED_MINTERS = [];  // list of pubkeys that are indeed allowed to mint

const { event, tagIndex } = JSON.parse(arguments[0]);  // decode the input argument, and extract event and tagIndex

const [ tagName ] = event.tags[tagIndex];  // extract the validator tag name

if (tagName !== "v") {  // (OPTIONAL) verify that we are indeed passed a validator tag
  return false;         // fail if we're not
}

var total = 0;  // keep running total of how many funds are created

for (const input of event.content.inputs) {                        // iterate through each input
    if (NOSTR.read([{"kind": 1001, "#y": input.id}]) != []) {      // verify that the input is not
        return false;                                              // already burnt, fail otherwise
    }
    const outputs = NOSTR.read([{"kind": 1001, "#z": input.id}]);  // retrieve all outputs associated to this input
    if (outputs.length != 1) {                                     // check there's only one,
        return false;                                              // fail otherwise
    }
    const [ output ] = outputs;                                     // keep said output
    if (output.destination != event.pubkey) {                      // verify it's directed to us,
        return false;                                              // fail otherwise
    }
    if (sha256toHex(input.nonce) != output.commitment) {           // verify it matches the commitment,
        return false;                                              // fail otherwise
    }
    total += output.quantity;                                      // accumulate running total
}

for (const output of event.content.outputs) {  // iterate through each output
    total -= output.quantity;                  // decrease running total
}

// if the net effect is either to burn or break-even in token quantities, return true,
// otherwise (ie. minting), check we are among the allowed minters
return total <= 0 || ALLOWED_MINTERS.includes(event.pubkey)
```

### Additional checks

1. If there are more outputs than inputs, new tokens are being minted.
2. If there are more inputs than outputs, then tokens are being burned.

Implementations can enforce further checks using these facts.

## Extensions

An output object may include a `.quantity` field, specifying how many fungible units of the token being minted this output is good for.
When used as input though, ALL such units are burned, and the user generating the event is responsible for sending the unused token units to themselves.

Needless to say, the checks need to take this new field into account when validating.
