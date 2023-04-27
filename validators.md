<!-- markdownlint-enable -->
<!-- markdownlint-disable MD013 -->

# NIP-XX --- Validators

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**,  **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

`kind:1111`
`tag:v` `tag:v-language` `tag:v-hints`

`depends:01` `depends:16`
`mentions:13` `mentions:09`

---

- [1. Motivation](#1-motivation)
- [2. Short Description](#2-short-description)
- [3. Overview](#3-overview)
- [4. Glossary](#4-glossary)
- [5. Validator Definition Event](#5-validator-definition-event)
- [6. Validator Tag](#6-validator-tag)
- [7. Validation](#7-validation)
  - [7.1. Unknown Validators](#71-unknown-validators)
  - [7.2. Invalid Validators](#72-invalid-validators)
  - [7.3. Runtime Context](#73-runtime-context)
    - [7.3.1. The `NostrRead` Capability](#731-the-nostrread-capability)
    - [7.3.2. The `NostrValidate` Capability](#732-the-nostrvalidate-capability)
- [8. Client Behavior](#8-client-behavior)
  - [8.1. Embedded Validators](#81-embedded-validators)
- [9. Use Cases](#9-use-cases)
  - [9.1. Oracles](#91-oracles)
  - [9.2. Validator Code Pinning](#92-validator-code-pinning)
  - [9.3. Userland NIP Implementations](#93-userland-nip-implementations)
    - [9.3.1. NIP-13: Proof of Work](#931-nip-13-proof-of-work)
  - [9.4. SNTs: Simple NOSTR Tokens](#94-snts-simple-nostr-tokens)
- [10. FAQ](#10-faq)
- [Appendixes](#appendixes)
  - [I. Implementation Considerations](#i-implementation-considerations)
  - [II. Future Work](#ii-future-work)
    - [II.1. VIPs](#ii1-vips)
    - [II.2. Validator Metadata](#ii2-validator-metadata)
  - [III. Recognized `v-language` Tags](#iii-recognized-v-language-tags)
    - [III.1. JavaScript](#iii1-javascript)

---

## 1. Motivation

The purpose of this NIP is to provide a framework for broadcasting immutable code blocks that will act as validators, and for specific events to declare which of these code blocks to use to validate itself.

## 2. Short Description

The underlying motivation is to bring a form of smart contracts to NOSTR, but the realities of a smart contract blockchain (eg. Rootstock, Ethereum, etc.) and NOSTR are quite different.

Firstly, we require the actual code of these smart contracts to be held somewhere.
Thus, we reserve a new kind to that effect and stipulate that the event's `.content` field must then contain the source code itself.
Since we don't want to restrict ourselves to a specific programming language, a new ("long-form") tag is defined that will declare the language used, and since languages can be further tailored by tweaking their parameters and execution environments, such tag is allowed to communicate those capabilities as well.

Now that we have a place to store our code, we need to find a way to actually use it.
The way in which we do this is by defining a new ("short-form") tag that will contain an event ID indicating the event where the code to execute resides (we will require the event with the ID mentioned in such tags to be of the newly reserved kind mentioned above).
One can have several such tags, indicating that several code blocks should be executed for this event.
Lastly, each tag can carry additional input parameters for the code block referred to, this is so that parameters intended for one code block but not another can have a place to rest.

Finally, we need to define how those code blocks are executed and what the effects of doing so are.
This is the trickiest part, since we allow for different source code languages, and each language has their specific quirks, thus, we'll need to define "execution conventions" for each language officially supported.
Whatever the specific conventions for each language are, we shall stipulate that the code block will receive the whole event and a tag index as input, and we expect it to return either **TRUE** or **FALSE**.
The significance of this return value will be: a **TRUE** value attests that the message is "valid", while a **FALSE** value declares it "invalid".
Thus, we shall call these code blocks **validators**.

Clients are free to ignore validator return value, call them discretionally, or not even call them at all, they're strictly optional and simply serve the purpose of stating that an event adheres to an unchanging validation algorithm.

This decentralized and asynchronous execution model does present some caveats though: since a validator's code could in principle be run several times by different users, validators need to be "read-only", ie. they should not cause any side effects, for they would be realized an unpredictable number of times.

## 3. Overview

This document is organized as follows: first we introduce some terms we shall use further down in a [Glossary](#4-glossary), next we describe the [Validator Definition Event](#5-validator-definition-event) this NIP introduces, along with its language tag, next the [Validator Tag](#6-validator-tag) used in events to be validated is presented, next the [Validation](#7-validation) procedure is explained, and finally [Client Behavior](#8-client-behavior) is discussed.
We close with proposed [Use Cases](#9-use-cases) and a [FAQ](#10-faq) section.
Finally, [Appendixes](#appendixes) follow to deal with technical aspects.

## 4. Glossary

**Validator:** a piece of code accepting a NOSTR event as input and returning either **TRUE** or **FALSE** (observing the conventions of the source code's language).

**Language Tag:** a NOSTR tag attached to a Validator Definition Event used to indicate the programming language used.

**Validator Definition Event:** a NOSTR event containing a Validator and its corresponding Language Tag.

**Validator Tag:** a NOSTR tag attached to an event indicating the Validator Definition Event to use.

**Validating:** the act of running a Validator against an event.

## 5. Validator Definition Event

A _validator definition event_ is defined as an event with `kind:1111`.

A validator definition event's `.tags` field **MUST** include ONE AND ONLY ONE `"v-language"` tag, conforming to the following format:

```json
[
  "v-language",
  <LANGUAGE>,
  <CAPABILITY>,
  ...
]
```

The `<LANGUAGE>` placeholder **MUST** be a string, and it **SHOULD** equal one of the ones listed in [Appendix III](#iii-recognized-v-language-tags), although clients can support languages not explicitly listed therein, and said list may be expanded upon at a later time.

The `<CAPABILITY>` placeholders **MAY** be omitted altogether if not needed, and they consist of an arbitrary number of arbitrary strings; if given, though, they **SHOULD** correspond to those specified in [Appendix III](#iii-recognized-v-language-tags) in accordance to the value of the `<LANGUAGE>` placeholder.

A validator definition event's `.content` field **MUST** contain source code expressed in the `<LANGUAGE>` specified in the `"v-language"` tag, possibly making use of any `<CAPABILITY>` provided.

A validator definition event's `.tags` field **MAY** contain ONE OR MORE `"v-hints"` tags, conforming to the following format:

```json
[
  "v-hints",
  <HINT>,
  ...
]
```

The `<HINT>` placeholder, if present, **SHOULD** be one of:

- **`"CACHE"`:** the presence of this hint lets the client know that the validator's result can safely be cached (ie. the same event ID will yield the same result),
- **`"FRESH"`:** the presence of this hint lets the client know that the validator's result can _NOT_ safely be cached (ie. the same event ID won't necessarily yield the same result),
- **`"LAZY"`:** this hint lets the client know that this validator should be executed solely in response to the end user asking for it,
- **`"EAGER"`:** this hint lets the client know that this validator should be executed as soon as the message is received if possible,

this list is not exhaustive and clients are free to support additional hints if they so wish.

Clients can take the following as guidelines regarding hints:

- a validator using _no_ capabilities can safely be assumed to be hinted with `"CACHE"` if not otherwise hinted with `"FRESH"`,
- a validator using either the `NostrRead` or `NostrValidate` capabilities can conservatively be assumed to be hinted with `"LAZY"`; clients are encouraged to ignore the `"EAGER"` hint for validators using either the `NostrRead` or `NostrValidate` capabilities, except when they have been previously vetted by the client implementation team.

> Note that according to [NIP-16](https://github.com/nostr-protocol/nips/blob/master/16.md) validator definition events should be stored and **MUST NOT** be replaced at all.

Finally, `kind:1111` validator definition events **MUST NOT** be deleted and both relays and clients **MUST** ignore `kind:5` events (ie deletion events, see [NIP-09](https://github.com/nostr-protocol/nips/blob/master/09.md)) referring to them.

## 6. Validator Tag

A _validator tag_ is a NOSTR tag using the single-letter **`"v"`** conforming to the following format:

```json
[
  "v",
  <VALIDATOR_DEFINITION_EVENT_ID>,
  <ADDITIONAL_ARGUMENT>,
  ...
]
```

The `<VALIDATOR_DEFINITION_EVENT_ID>` **MUST** belong to an event of `kind:1111`.

The `<ADDITIONAL_ARGUMENT>` values **MAY** be omitted altogether if not needed.

Any number of validator tags can be attached to an event.

## 7. Validation

Validating an event consists of retrieving all of its validator tags and executing the corresponding validators.
To do this, clients should query for the event ID mentioned in the validator tag, read the `"v-language"` tag to ensure support (both for the language and the capabilities mentioned therein), and load the `.content` field as source code to be executed.

With the set up taken care of, the source code will get passed a JSON string of the form:

```json
{
  "event": <EVENT_TO_VALIDATE>,
  "tagIndex": <VALIDATOR_TAG_INDEX>,
  ...
}
```

the `.event` and `.tagIndex` fields are **REQUIRED**, but clients may include additional fields.

The value of the `<EVENT_TO_VALIDATE>` placeholder must consist of the event being validated (ie. the one sporting the `"v"` tag).
The value of the `<VALIDATOR_TAG_INDEX>` placeholder must consist of an integer indicating the index (0 based) of the tag that triggered this particular validation (ie. the index into the event's `.tag` field where the validator can find the `"v"` tag that corresponds to this validation effort).

The validator source code is now run and its return value obtained: if the return value represents a **TRUE** value, the event is said to have _passed_ validation, if the return values represents a **FALSE** value, the event is said to have _failed_ validation.

If all validators specified via tags have been executed and passed, the whole event is said to have passed validation, otherwise, the event as a whole is said to have failed validation.

### 7.1. Unknown Validators

If the validator tag refers to an irretrievable event ID, that specific validation is said to be _unknown_.
An event whose validator tags have all either passed validation or found to be unknown is said to have _incomplete_ validation.

Clients are free to determine how such an event must be handled, they can either consider incomplete validation to be a failure or a success.

### 7.2. Invalid Validators

If the validator tag refers to an event not of `kind:1111`, or contains an invalid `"v-language"` tag, that validator tag is said to be invalid and considered to have _failed_ validation.

### 7.3. Runtime Context

During the execution of the validator code proper, clients **MAY** provide additional capabilities for the code to use.
These can range from utility libraries (eg. JSON parsing utilities), to external communication facilities (eg. querying [IPFS](https://docs.ipfs.tech/)), to specific language conventions being enabled or not (eg. feature flags for experimental language features).

Although specific `"v-language"` conventions can declare a capability to be realized in any specific manner, extensions **SHOULD** adhere to the following guidelines:

1. Functionality _not_ provided by the programming language in a "standard" fashion **SHOULD** be externalized to a capability.
2. Functionality that exhibits a _non-idempotent_ behavior **SHOULD** be externalized to a capability.

Discretion is afforded to extension authors regarding what precisely should be considered _a "standard" fashion_, but as a general rule, standard libraries and _de facto_ standards are understood to fit the description.

#### 7.3.1. The `NostrRead` Capability

Clients **MUST** provide validators with a NOSTR querying facility identified with `NostrRead` that will accept a [`REQ` filter specification](https://github.com/nostr-protocol/nips/blob/master/01.md#from-client-to-relay-sending-events-and-creating-subscriptions) and an optional relay URL, and query said relay to retrieve the results matching said filters without establishing a subscription.
The pseudocode for such a capability call would look like:

```text
NostrRead((<FILTER>, ... ))
```

(where additional filters are optional), without an explicit relay, or

```text
NostrRead((<FILTER>, ... ), <RELAY_URL>)
```

passing an explicit relay URL.
The return value would look like:

```text
(
  <EVENT>,
  ...
)
```

Of course, each validator language will demand their own calling conventions and specifics, as will specify the actual result values.
Note that specific languages **MAY NOT** chose to not provide this specific capability: it is required to exist for them all in one form or another.
In [Appendix III](#iii-recognized-v-language-tags), each recognized language will specify how to access this capability.

This capability allows for validators to perform introspection on the NOSTR network.

#### 7.3.2. The `NostrValidate` Capability

Clients **MUST** provide validators with a NOSTR validation facility identified with `NostrValidate` that will accept a NOSTR event and execute all validators defined for it, returning **TRUE** if the event passes validation, or **FALSE** otherwise.
The pseudocode for such a capability call would look like:

```text
NostrValidate(event)
```

The return value would simply consist of the resulting boolean value.

Of course, each validator language will demand their own calling conventions and specifics, as will specify the actual result values.
Note that specific languages **MAY NOT** chose to not provide this specific capability: it is required to exist for them all in one form or another.
In [Appendix III](#iii-recognized-v-language-tags), each recognized language will specify how to access this capability.

This capability allows for validators to perform cascaded validation on queried events.

> **NOTE:** the usage of this capability runs the risk of taking longer times to validate.
> Clients are advised to _not_ run validators making use of `NostrValidate` capabilities in an eager fashion, and only do so if at all for whitelisted validator IDs.

## 8. Client Behavior

Supporting clients **MAY** validate incoming events, and they may do so either _eagerly_ or _lazily_ upon the user's request.

When validating, [Invalid Validators](#72-invalid-validators) should be treated as failing validation, and [Unknown Validators](#71-unknown-validators) should be flagged and the user should decide what should be done then.

### 8.1. Embedded Validators

Clients are not compelled to execute the validator in precisely the fashion outlined above, but can rather do so in any way functionally equivalent to it.

This means that whatever execution strategy that would result in exactly the same validation status can be applied by clients.

As more and more [validator definition events](#5-validator-definition-event) are referenced in [validator tags](#6-validator-tag), a form of soft general consensus will organically emerge, with certain validators being found to be especially useful or ubiquitous; clients **MAY** then implement validation routines that will effectively implement the same validation procedure, and have them trigger when the corresponding validator event ID is found in a `"v"` tag.
This allows clients to avoid loading the code from the [validator definition event](#5-validator-definition-event) and undergo JSON (de)serialization, and instead run their internal corresponding validation routines for increased performance.

This particular strategy of implementing validators internally by clients is termed _embedded validation_.

## 9. Use Cases

In what follows, we'll look at some use cases for NOSTR validators.

### 9.1. Oracles

NOSTR can be greatly extended by providing oracles tying an event's validity to the outside world (care must be taken though to prevent the result from one such validation from changing in time, since validation occurs multiple times, it would lead to events being valid or invalid depending on the moment in time in which they occur... on the other hand, ephemeral events could very well use this very property... the mind boggles).

By way of example, we present here a validator that will ensure the Bitcoin network has reached the given block height:

```javascript
// Requires the "XMLHttpRequest" and "Async" capabilities

const { event, tagIndex } = JSON.parse(arguments[0]);  // decode the input argument, and extract event and tagIndex

const [ tagName, , expectedHeight ] = event.tags[tagIndex];  // extract the validator tag name and expected height from the event

if (tagName !== "v") {  // (OPTIONAL) verify that we are indeed passed a validator tag
  return false;         // fail if we're not
}

const url = `https://blockchain.info/block-height/${expectedHeight}`;  // the block-height query URL

const request = new XMLHttpRequest();                      // build a new XMLHttpRequest
request.open("GET", url, false);                           // set up a synchronous GET request to the above URL
request.send();                                            // execute it
return JSON.parse(request.responseText)["blocks"] !== [];  // compare it against the expected error response
```

One would use such a validator with the given validator tag:

```json
[
  "v",
  <VALIDATOR_ID>,
  <EXPECTED_BLOCK_HEIGHT>
]
```

### 9.2. Validator Code Pinning

Validator Code Pinning refers to the act of storing the same validator code in more than one place, and having a validator check that those two places do indeed contain the same code.

This makes it easier to interoperate, since the code itself can be held at a "customary" location (GitHub, GitLab, IPFS, etc.), while the actual validator definition event lives in the NOSTR network.
A validator performing code pinning would take the "customary" location as an additional parameter, and would be applied to events of `kind:1111` (ie. to validators themselves).

One such validator can be very simply implemented:

```javascript
// Requires the "XMLHttpRequest" and "Async" capabilities

const { event, tagIndex } = JSON.parse(arguments[0]);  // decode the input argument, and extract event and tagIndex

const [ tagName, , canonicalUrl ] = event.tags[tagIndex];  // extract the validator tag name and canonical content URL from the event

if (tagName !== "v") {  // (OPTIONAL) verify that we are indeed passed a validator tag
  return false;         // fail if we're not
}
if (event.kind !== 1111) {  // verify that we are indeed validating a validator
  return false;             // fail if we're not
}

const request = new XMLHttpRequest();           // build a new XMLHttpRequest
request.open("GET", canonicalUrl, false);       // set up a synchronous GET request to the above URL
request.send();                                 // execute it
return request.responseText === event.content;  // compare it against the event's content
```

In order to use this validator you can attach the following validator tag:

```json
[
  "v",
  <VALIDATOR_ID>,
  <CANONICAL_URL>
]
```

### 9.3. Userland NIP Implementations

Some NIPs can be implemented via validators, this shows that implementing this NIP could transfer protocol maintainability and specialization to the user base, without losing the unicity of specification since validator definition events are immutable.

#### 9.3.1. NIP-13: Proof of Work

A [NIP-13](https://github.com/nostr-protocol/nips/blob/master/13.md) Proof-of-Work validator can be very simply coded thusly:

```javascript
// Requires the "Crypto" and "Async" capabilities

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

const { event, tagIndex } = JSON.parse(arguments[0]);  // decode the input argument, and extract event and tagIndex

const [ tagName ] = event.tags[tagIndex];  // extract the validator tag from the event

if (tagName !== "v") {  // (OPTIONAL) verify that we are indeed passed a validator tag
  return false;         // fail if we're not
}

// calculate the event ID according to NIP-01
const calculatedId = await sha256toHex(
  JSON.stringify([
    0,
    event.pubkey,
    event.created_at,
    event.kind,
    event.tags,
    event.content
  ])
);

if (calculatedId !== event.id) {  // if the calculated ID and the event ID aren't equal,
  return false;                   // then fail
}

const difficulty = event.tags         // although NIP-13 is unclear as to how to manage
  .filter(tag => tag[0] === "nonce")  // multiple "nonce" tags, we take the conservative
  .map(tag => tag[2])                 // approach and consider multiple "nonce" tags as
  .reduce(Math.max)                   // describing differing levels of difficulty,
;                                     // keeping only the highest of them

var num0s = 0;  // the number of leading 0s

// break the ID into 32-bit blocks and fast-forward the count as long as they are 0
for (let i = 0; i < 32; i += 8) {
  const currentInt = parseInt(event.id.substring(i, i + 8), 16);
  num0s += Math.clz32(currentInt);
  if (!currentInt) {
    break;
  }
}

return difficulty <= num0s;  // validate that the number of leading 0s is at least the difficulty
```

This validator can be used by simply mentioning the validator ID since all the data required for it to work is already present in the event itself:

```json
[
  "v",
  <VALIDATOR_ID>
]
```

### 9.4. SNTs: Simple NOSTR Tokens

A very simple Bitcoin-inspired token system can be implemented via validators.

We'll use `kind:1001` as an example in what follows for the _token flow events_ moving tokens around.
Likewise, we'll use tags `"y"` and `"z"` for _input token IDs_ and _output token IDs_.

Tokens are abstract entities represented by a UUIDv4 (its ID).
In order to transfer a token, it needs to be "burned" and a new token created in the same flow event.
You can think of tokens as 1-Satoshi Bitcoin UTXOs.

A token flow event has the following form:

```json
{
  ...,
  "kind": 1001,
  ...,
  "pubkey": <PUBKEY>,
  ...,
  "tags": [
    ...,
    ["y", <TOKEN_ID>],  // used input IDs
    ...,
    ["z", <TOKEN_ID>],  // used output IDs
    ...
    ["p", <DESTINATION>],  // mentioned destination pubkeys --- NOTE: the recommended relay URL, if given, is ignored
    ...
  ],
  ...,
  "content": <CONTENT>,  // the JSON string serialization of the content object detailed below
  ...
}
```

where the content follows:

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
      "destination": <DESTINATION>   // PubKey --- the PubKey of the output's destination,
    },
    ...
  ]
}
```

All input IDs mentioned in `.inputs.*.id` must appear in `"y"` tags.

All output IDs mentioned in `.outputs.*.id` must appear in `"z"` tags.

All pubkeys mentioned in `.outputs.*.destination` must appear in `"p"` tags.

A validator for Token Flow events follows:

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
  const filter = [
    {
      "kinds": [1001],
      `#${tagName}`: ids,
      "until": until,
    }
  ];

  var entries = {};
  for (const relay of RELAYS) {
    for (const event of nostrReadValidated(filter, relay)) {
      if (!entries.has(event.id)) {
        entries[event.id] = {"count": 0, "event": event};
      }
      entries[event.id].count++;
    }
  }

  const limit = RELAYS.length >> 1;

  const deletionFilter = [
    {
      "kinds": [5],
      `#e`: [
        entries
          .values()
          .filter(entry => limit < entry.count)
          .map(entry => entry.event.id)
      ],
      "until": until,
    }
  ];
  for (const relay of RELAYS) {
    for (const event of nostrReadValidated(deletionFilter, relay)) {
      entries[event.id].count--;
    }
  }

  return entries
    .values()
    .filter(entry => limit < entry.count)
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
  return false;             // fail if we're not
}                           //
if (event.kind !== 1001) {  // verify that we're being run on a Token Flow event
  return false;             // fail if we're not
}                           //

const taggedInputs = tagValues("y", event);        // retrieve the set of all input tags
const taggedOutputs = tagValues("z", event);       // retrieve the set of all output tags
const taggedDestinations = tagValues("p", event);  // retrieve the set of all destination tags

var seenInputs = new Set();        // initialize seen inputs
var seenOutputs = new Set();       // initialize seen outputs
var seenDestinations = new Set();  // initialize seen destinations

const content = JSON.parse(event.content);  // parse the event's content

var total = 0;  // keep running total of how many funds are moved

for (const input of content.inputs) {                          // iterate through each input
  if (seenInputs.has(input.id)) {                              // verify the input ID is not repeated
    return false;                                              // fail if it is
  }                                                            //
  if ([] !== fetchInputs([input.id], event.created_at)) {      // verify that the input is not
    return false;                                              // already burnt, fail otherwise
  }                                                            //
  const outputs = fetchOutputs([input.id], event.created_at);  // retrieve all outputs associated to this input
  if (1 !== outputs.length) {                                  // verify there's only one of them
    return false;                                              // fail otherwise
  }                                                            //
  if (output[0].destination != event.pubkey) {                 // verify it's directed to us,
    return false;                                              // fail otherwise
  }                                                            //
  if (sha256toHex(input.nonce) != output[0].commitment) {      // verify it matches the commitment,
    return false;                                              // fail otherwise
  }                                                            //
  total++;                                                     // accumulate running total
  seenInputs.add(input.id);                                    // add the current input ID to the seen ones
}                                                              //

for (const output of content.outputs) {      // iterate through each output
  if (seenOutputs.has(output.id)) {          // verify the output ID is not repeated
    return false;                            // fail if it is
  }                                          //
  total--;                                   // decrease running total
  seenOutputs.add(output.id);                // add the current output ID to the seen ones
  seenDestinations.add(output.destination);  // add the destination ID to the seen ones
}                                            //

if (fetchOutputs(Array.from(seenOutputs), event.created_at) !== []) {  // make sure the newly-created outputs are new
  return false;                                                        // fail otherwise
}                                                                      //

if (!equalSets(taggedInputs, seenInputs)) {              // check that all seen inputs are tagged
  return false;                                          // fail if not
}                                                        //
if (!equalSets(taggedOutputs, seenOutputs)) {            // check that all seen outputs are tagged
  return false;                                          // fail if not
}                                                        //
if (!equalSets(taggedDestinations, seenDestinations)) {  // check that all seen destinations are tagged
  return false;                                          // fail if not
}                                                        //

if (total < 0) {                                                    // if we're minting tokens
  return Array.from(ALLOWED_MINTERS)                                // check that at least one allowed minter
    .some(allowedMinter => event.pubkey.startsWith(allowedMinter))  // is a prefix of the current pubkey
  ;                                                                 //
} else if (0 < total) {                                             // if we're burning tokens
  return BURN;                                                      // this is valid only if burning allowed
} else {                                                            // otherwise
  return true;                                                      // everything looks fine, validate
}                                                                   //
```

> **NOTE:** although care has been taken when writing this validator, it should go without saying that this is merely an example and **NOT** intended to be used in any production capacity whatsoever.

As the docblock in the validator proper reads, the `BURN`, `RELAYS`, `PRE_MINTED`, and `ALLOWED_MINTERS` can be tweaked to realize several different token policies.

One possible usage of these simple tokens would be to provide ownership attestation: showing that certain token IDs are indeed in possession of the event signer.
One such use case can in turn be realized by the following validator:

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
 * required to be in sync with the one in the corresponding NOSTR Tokens Validator.
 *
 * Usage merely requires adding a tag of the form:
 *
 *     ["v", "<VALIDATOR_MESSAGE_ID>", "<TOKEN_ID_1>", "<TOKEN_ID_2>", ..., "<TOKEN_ID_N>", ...]
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
  const filter = [
    {
      "kinds": [1001],
      `#${tagName}`: ids,
      "until": until,
    }
  ];

  var entries = {};
  for (const relay of RELAYS) {
    for (const event of nostrReadValidated(filter, relay)) {
      if (!entries.has(event.id)) {
        entries[event.id] = {"count": 0, "event": event};
      }
      entries[event.id].count++;
    }
  }

  const limit = RELAYS.length >> 1;

  const deletionFilter = [
    {
      "kinds": [5],
      `#e`: [
        entries
          .values()
          .filter(entry => limit < entry.count)
          .map(entry => entry.event.id)
      ],
      "until": until,
    }
  ];
  for (const relay of RELAYS) {
    for (const event of nostrReadValidated(deletionFilter, relay)) {
      entries[event.id].count--;
    }
  }

  return entries
    .values()
    .filter(entry => limit < entry.count)
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

const [ tagName, , ...tokenIds ] = event.tags[tagIndex];  // extract the validator tag name and token IDs

if (tagName !== "v") {  // (OPTIONAL) verify that we are indeed passed a validator tag
  return false;         // fail if we're not
}                       //

const tokenIdsClean = Set(tokenIds);                      // remove duplicates from input token IDs
const outputs = fetchOutputs(                             // retrieve all outputs generating these
    Array.from(tokenIdsClean),                            // token IDs
    event.created_at                                      //
  )                                                       //
  .filter(output => output.destination === event.pubkey)  // keep only those that come our way
;                                                         //

if (outputs.length !== tokenIdsClean.length) {                          // verify we have them all
  return false;                                                         // fail otherwise
}                                                                       //
if (fetchInputs(Array.from(tokenIdsClean), event.created_at) !== []) {  // verify no inputs are used
  return false;                                                         // fail otherwise
}                                                                       //

return true;  // if we got here, everything is fine
```

> **NOTE:** although care has been taken when writing this validator, it should go without saying that this is merely an example and **NOT** intended to be used in any production capacity whatsoever.

## 10. FAQ

**Why use a single-letter tag (ie. `"v"`) for validator tags?**

The reason behind this is twofold:

- on the one hand, this allows for clients to build a `REQ` query restricting themselves to events validated by specific validators, improving efficiency on the client side,
- on the other hand, it allows validators to be freely composed via the `NostrRead` capability: a validator can look for events referring the one being validated that are themselves validated by a specific validators.

Were we not to use a single-letter tag, filtering out the results client-side could be time consuming and cumbersome.

**Why does the validator get passed the tag index?**

This is so so that a validator can "find itself" in the event being validated.
By providing the tag's index, the validator can look into the event's tags for the one being processed, extract the associated event ID as its own (if needed), and pick up the additional arguments therein.

**What's the use of the validator tag `<ADDITIONAL_ARGUMENT>` placeholders?**

This allows a client to construct an event that will forward different parameters to different validators, were we not to have this, we would need to find another place to put these and this would require either encroaching into the `.content` field or defining a new field to hold these (or have validators "bake-in" all the required parameters, which would result in an explosion of algorithmically identical validators differing solely on actual arguments passed).

**Can the same validator be specified more than once?**

Indeed, but doing so without varying the additional arguments given in each case would be pointless, and can in fact be optimized away by complying clients.

**Can the validators be run in parallel?**

Absolutely.
Validators **SHOULD** be independent in the sense that they should cause no state change whatsoever other than those implied by an event being valid or not.

## Appendixes

The appendixes that follow deal with technical and governance aspects of the proposal.

### I. Implementation Considerations

Clients choosing to implement validation would do well to take defensive measures when running unknown validators.

Although the purpose of this NIP is not to provide exact implementation guidelines, these specific defensive strategies are too obvious not to mention:

- **Isolation:** clients should run validator code in as an isolated environment as possible; this is to prevent validator code from leaking into the client proper and interfering with its operation.
- **Timeout:** clients should set a form of timeout for the running of each validator; this would reduce the exposed area and help mitigate any runaway validator code.
- **Warding:** validators that connect to the outside world via communication capabilities should be "warded" and the specific communication counterparts (ie. contacted hosts for instance) curated and whitelisted; this will keep control of untrusted access in the hands of the client, preventing potential exfiltration of sensitive data.

It should be noted that when clients implement [embedded validation](#81-embedded-validators), most of these strategies are no longer required, since it is understood that client developers have curated the validators to embed so as to make any concurrent defense redundant (save, perhaps, for the warding of outside world communication).

### II. Future Work

No single NIP can cover every possible caveat that may come up in a setting such as this.
Several future work avenues remain.

In what follows, we investigate just some of them.

#### II.1. VIPs

As NOSTR has NIPs as its underlying governance mechanism, validators require their own as well.
This is not to eclipse the NIPs mechanism, but rather to increase interoperability between clients, especially in light of [embedded validation](#81-embedded-validators).

As validators settle and get embedded in clients, bugs, enhancements, or mere changes to well-established ones will will need to be taken care of.
Although there's no need to define a governance mechanism in its entirety here, we do want to provide some guidelines any such mechanism should follow:

- **Proposal distribution:** VIP documents should be distributed in a resilient and decentralized manner, ideally utilizing the NOSTR network itself (perhaps via a [NIP-23](https://github.com/nostr-protocol/nips/blob/master/23.md) event).
- **Proposal discussion and validation:** discussion of the proposal should be conducted in an open and transparent manner, ideally within the NOSTR network itself (perhaps via comments to the original VIP document).
- **Proposal voting and acceptance:** clients and end users should then vote on the VIP proposal, voting should be transparent and open, ideally tracked in the NOSTR network itself as well.

#### II.2. Validator Metadata

Mechanisms for associating semantically-specific metadata to validators need to eventually be provided.
Although very many such metadata can be conceived of, the following two are immediately useful:

- **Test Vectors:** test vectors are events containing examples of both positive and negative validation instances for a specific validator; these act as guides any conforming implementation of said validator must follow; they help in providing a complete test suite for [embedded validators](#81-embedded-validators).
- **Equivalence Assertions:** two validators may effectively consist of differing implementations of the same underlying validation algorithm, when this happens, equivalence assertions effectively state that two validators are indeed the same for all practical purposes; this is especially useful, again, in implementing [embedded validation](#81-embedded-validators), as it allows conforming clients to run the embedded versions of equivalent validators even when the mentioned IDs are not exactly the ones originally programmed in.

### III. Recognized `"v-language"` Tags

The current NIP recognizes the following languages and their corresponding parameters for usage in the `"v-language"` tag.

#### III.1. JavaScript

The `<LANGUAGE>` placeholder **MUST** be `"javascript"`.

The validator definition event's `.content` **MUST** be ES6-compliant.

Return values are interpreted as JavaScript booleans.
Uncaught exceptions are considered `false` return values.

The execution environment will vary depending on the actual type of execution machine (ie. NodeJS vs browser-based), but the execution tactic should be compatible with extracting the value of the following expression:

```javascript
try {
  return (new Function(validatorEvent.content)).apply(
    {},
    JSON.stringify([event, tagIndex]),
  );
} catch {
  return false;
}
```

where `event` and `tagIndex` are as above, and `validatorEvent` is the validator definition event referred to by ID in the event's `"v"` tag.

The [`DefaultLocale()`](https://402.ecma-international.org/4.0/#sec-defaultlocale) abstract operation **MUST** return the `C` locale.

No support is explicitly provided for JavaScript Events, but conforming clients are free to define a capability (perhaps called `Events`) to signal usage and support for this functionality.

The **`NostrRead`** capability is implemented via a global class `NOSTR`, containing methods:

- **`NOSTR.read(filters)`:** where `filters` is an array of objects realizing the filter specification required.
- **`NOSTR.read(filters, relayUrl)`:** where `filters` is an array of objects realizing the filter specification required, and `relayUrl` is a string containing the relay URL to use.

The **`NostrValidate`** capability is implemented via a global class `NOSTR`, containing method:

- **`NOSTR.validate(event)`:** where `event` is an object containing a NOSTR event.

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
| `AsyncFunction`                                             | `Async`          | -            |
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
