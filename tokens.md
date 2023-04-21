<!-- markdownlint-enable -->
<!-- markdownlint-disable MD013 -->

# NOSTR Tokens

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
        ["t", <INPUT_ID>, "in"],
        ...,
        ["t", <OUTPUT_ID>, "out"],
        ...
    ],
    ...,
    "content": {
        "inputs": [
            ...,
            {
                "id" <ID>,   // UUIDv4 --- the ID of an unburned output
                "nonce": <NONCE>  // NONCE --- the NONCE of said output, such that SHA3(NONCE) == Output(ID).commitment
            }
            ...
        ],
        "outputs": [
            ...,
            {
                "id": <ID>,                   // UUIDv4 --- a random ID to associate to this output
                "commitment": <COMMITMENT>,   // SHA3 of NONCE --- public commitment to the value of NONCE
                "secret": <SECRET>,             // Encrypt(DESTINATION, NONCE) --- private revelation of the value of NONCE
                "destination": <DESTINATION>  // PubKey --- the PubKey of the output's destination
            },
            ...
        ]
    },
    ...
}
```

## Checks

1. All `.content.inputs.*.id` values should correspond to existing `.content.outputs.*.id` values, for which there's no other `kind:1001` message mentioning them in their own `.content.inputs.*.id` values.
2. All such `.content.outputs.*.id` values should have `PUBKEY` as their corresponding `.content.outputs.*.destination` values.
3. For each `.content.inputs.*.id`, check that applying `SHA3` to the corresponding `.content.inputs.*.nonce` value equals the corresponding `.content.outputs.*.commitment` value.

### Additional checks

1. If there are more outputs than inputs, new tokens are being minted.
2. If there are more inputs than outputs, then tokens are being burned.

Implementations can enforce further checks using these facts.

## Extensions

An output object may include a `.quantity` field, specifying how many fungible units of the token being minted this output is good for.
When used as input though, ALL such units are burned, and the user generating the event is responsible for sending the unused token units to themselves.

Needless to say, the checks need to take this new field into account when validating.
