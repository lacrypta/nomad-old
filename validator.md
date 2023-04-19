# NIP-XX

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**,  **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

`kind:1111`
`tag:v` `tag:v-language`

`depends:01` `depends:16`
`mentions:13`

---

- [1. Motivation](#1-motivation)
- [2. Short Description](#2-short-description)
- [3. Overview](#3-overview)
- [4. Glossary](#4-glossary)
- [5. Validator Event](#5-validator-event)
- [6. Validator Tag](#6-validator-tag)
- [7. Validation](#7-validation)
  - [7.1. Unknown Validators](#71-unknown-validators)
  - [7.2. Invalid Validators](#72-invalid-validators)
  - [7.3. Runtime Context](#73-runtime-context)
    - [7.3.1. The `NostrRead` Capability](#731-the-nostrread-capability)
- [8. Relay Behavior](#8-relay-behavior)
  - [8.1. NIP-11 Extra Fields](#81-nip-11-extra-fields)
  - [8.2. NIP-20 Command Results](#82-nip-20-command-results)
- [9. Client Behavior](#9-client-behavior)
- [10. Examples](#10-examples)
- [11. Use Cases](#11-use-cases)
  - [11.1. Oracles](#111-oracles)
  - [11.2. Validator Code Pinning](#112-validator-code-pinning)
  - [11.3. Userland NIP Implementations](#113-userland-nip-implementations)
    - [11.3.1. NIP-13: Proof of Work](#1131-nip-13-proof-of-work)
- [12. FAQ](#12-faq)
- [13. Appendixes](#13-appendixes)
  - [13.1. Recognized `v-language` Tags](#131-recognized-v-language-tags)
    - [13.1.1. JavaScript](#1311-javascript)
    - [13.1.2 Lua](#1312-lua)

---

## 1. Motivation

The purpose of this NIP is to provide a framework for broadcasting immutable code blocks that will act as validators, and for specific events to declare which of these code blocks to use to validate itself.

## 2. Short Description

The underlying motivation is to bring a form of smart contracts to NOSTR, but the realities of a smart contract blockchain (eg. Ethereum) and NOSTR are quite different.

Firstly, we require the actual code of these smart contracts to be held somewhere.
Thus, we reserve a new kind to that effect and stipulate that the `content` field must then contain the source code itself.
Since we don't want to restrict ourselves to a specific programming language, a new ("long-form") tag is defined that will declare the language used, and since languages can be further tailored by tweaking their parameters and execution environments, such tag is allowed to communicate those capabilities as well.

Now that we have a place to store our code, we need to find a way to actually use it.
The way in which we do this is by defining a new ("short-form") tag that will contain an event ID indicating the event where the code to execute resides (we will require the event ID mentioned in such tags to be of the newly reserved kind mentioned above).
One can have several such tags, indicating that several code blocks should be executed for this event, and they will be executed in the order given.
Lastly, each tag can carry additional input parameters for the code block referred to, this is so that parameters intended for one code block but not another can have a place to rest.

Finally, we need to define how those code blocks are executed and what the effects of doing so are.
This is the trickiest part, since we allow for different source code languages, and each language has their specific quirks, thus, we'll need to define "execution conventions" for each language officially supported.
Whatever the specific conventions for each language are, we shall stipulate that the code block will receive the whole event as input, and we expect it to return either `true` or `false`.
The significance of this return value will be: a `true` value attests that the message is "valid", while a `false` value declares it "invalid".
Thus, we shall call these code blocks **validators**.

Clients are free to ignore validator return value, call them discretionally, or not even call them at all, they're strictly optional and simply serve the purpose of stating that an event adheres to an unchanging validation algorithm.

This decentralized and asynchronous execution model does have some caveats though: since a validator's code could in principle be run several times by different users, validators need to be "read-only", ie. they should not cause any side effects, for they would be realized an unpredictable number of times.

## 3. Overview

This document is organized as follows: first we introduce some terms we shall use further down in a Glossary, next we describe the Validator Event this NIP introduces, along with its language tag, next the Validator Tag used in events to be validated is presented, next the Validation procedure is explained, and finally Relay Behavior and Client Behavior are discussed.
We close with Examples, proposed Use Cases and a FAQ section.
Finally, appendixes follow to deal with technical aspects.

## 4. Glossary

**Agent:** either a NOSTR Client or NOSTR Relay.

**Validator:** a piece of code accepting a NOSTR event as input and returning either `true` or `false` (observing the convention of the source code's language).

**Language Tag:** a NOSTR tag attached to a Validator Event used to indicate the programming language used.

**Validator Event:** a NOSTR event containing a Validator and its corresponding Language Tag.

**Validator Tag:** a NOSTR tag attached to an event indicating the Validator Event to use.

**Validating:** the act of running a Validator against an event.

## 5. Validator Event

A _validator event_ is defined as an event with kind `1111`.

A validator event's `tags` field **MUST** include ONE AND ONLY ONE `v-language` tag, conforming to the following format:

```json
[
  "v-language",
  <LANGUAGE>,
  <CAPABILITY>,
  ...
]
```

The `<LANGUAGE>` placeholder **MUST** be a string, and it **SHOULD** equal one of the ones listed in [Appendix 13](#131-recognized-v-language-tags); although clients and relays can support languages not explicitly listed therein, and said list may be expanded upon at a later time.

The `<CAPABILITY>` placeholders **MAY** be omitted altogether if not needed, and they consist of an arbitrary number of arbitrary strings; if given, though, they **SHOULD** correspond to those specified in [Appendix 13](#131-recognized-v-language-tags) in accordance to the value of the `<LANGUAGE>` placeholder.

A validator event's `content` field **MUST** contain source code expressed in the `<LANGUAGE>` specified in the `v-language` tag, possibly making use of any `<CAPABILITY>` provided.

Note that according to [NIP-16](https://github.com/nostr-protocol/nips/blob/master/16.md) validator events should be stored and **MUST NOT** be replaced at all.

## 6. Validator Tag

A _validator tag_ is a NOSTR tag using the single-letter **`v`** conforming to the following format:

```json
[
  "v",
  <VALIDATOR_EVENT_ID>,
  <ADDITIONAL_ARGUMENT>,
  ...
]
```

The `<VALIDATOR_EVENT_ID>` **MUST** belong to an event of kind `1111`.

The `<ADDITIONAL_ARGUMENT>` values **MAY** be omitted altogether if not needed.

More than one validator tag can be attached to an event.

## 7. Validation

Validating an event consists of retrieving all of its validator tags and executing the corresponding validators in the order they appear.
To do this, agents should query for the event ID mentioned in the validator tag, read the `v-language` tag to ensure support, and load the `content` field as source code to be executed.

With the set up taken care of, the source code will get passed the following values in order:

1. the whole event being validated,
2. an index (0 based) indicating the tag number that triggered this particular validation invocation.

The validator source code is now run and its return value obtained: if the return value represents a **TRUE** value, the event is said to have _passed_ validation, if the return values represents a **FALSE** value, the event is said to have _failed_ validation.

If all validator tags have been executed and passed, the whole event is said to have passed validation, otherwise, the event as a whole is said to have failed validation.

In order to pass the event and tag index to the validator, and retrieve the validator's result, conversion procedures need to be defined for each supported language (see [Appendix 13](#131-recognized-v-language-tags) for details regarding the ones defined in this NIP).

### 7.1. Unknown Validators

If the validator tag refers to an irretrievable event ID, that specific validation is said to be _unreachable_. An event whose validator tags have all either passed validation or found to be unreachable is said to have _incomplete_ validation.

Agents are free to determine how such an event must be handled, they can either consider incomplete validation to be a failure or a success.

### 7.2. Invalid Validators

If the validator tag refers to an event not of kind `1111`, or contains an invalid `v-language` tag, that validator tag is said to be invalid and considered to have _failed_ validation.

### 7.3. Runtime Context

During the execution of the validator code proper, agents **MAY** provide additional capabilities for the code to use.
These can range from utility libraries (eg. JSON parsing utilities) to external communication facilities (eg. querying [IPFS](https://docs.ipfs.tech/)).

Although specific `v-language` conventions can declare a capability to be realized in any specific manner, extensions **SHOULD** adhere to the following guidelines:

1. Functionality _not_ provided by the programming language in a "standard" fashion **SHOULD** be externalized to a capability.
2. Functionality that exhibits a _non-idempotent_ behavior **SHOULD** be externalized to a capability.

Discretion is afforded to extension authors regarding what precisely should be considered _a "standard" fashion_, but as a general rule, standard libraries and _de facto_ standards are understood to fit the description.

#### 7.3.1. The `NostrRead` Capability

Agents **MUST** provide validators with a NOSTR querying facility identified with `NostrRead` that will accept a [`REQ` filter specification](https://github.com/nostr-protocol/nips/blob/master/01.md#from-client-to-relay-sending-events-and-creating-subscriptions) and retrieve the results without establishing a subscription.
The pseudocode for such a capability call would look like:

```text
NostrRead(
  <FILTER>,
  ...
)
```

(where additional filters are optional) and the return value would look like:

```text
(
  <EVENT>,
  ...
)
```

Of course, each validator language will demand their own calling conventions and specifics, as will specify the actual result values.
Note that specific languages **MAY NOT** chose to not provide this specific capability: it is required to exist for them all in one form or another.

This capability allows for validators to perform introspection on the NOSTR network.

## 8. Relay Behavior

Validation is completely optional for relays, they're free to ignore it and forward events along without validating.

Having said that, relays **MAY** opt to validate incoming events.
If so doing, they're free to apply whatever policy they see fit; they may for example only execute certain "known" validators, require a POW field to execute, demand payment in exchange for execution, or just run simple validations ensuring that validator tags are themselves valid.

If, however, a relay DOES opt to validate an incoming event, it **MUST** adhere to the following rules:

1. Upon encountering an [Invalid Validator](#72-invalid-validators) the relay **MUST** drop the event and not forward it.
2. Upon encountering an [Unknown Validator](#71-unknown-validators) the relay **MUST NOT** drop the event and **SHOULD** forward it.
3. If any of the validator tags the relay chooses to execute fails, the relay **MUST** drop the event and not forward it.

### 8.1. NIP-11 Extra Fields

Relays choosing to validate incoming events **MAY** advertise so in the response of a [NIP-11](https://github.com/nostr-protocol/nips/blob/master/11.md) query.

This entails adding the current NIP number to the `supported_nips` field and possibly including the following extra field in the response:

```json
{
  ...
  "validation": {
    "whitelisted": [
      <VALIDATOR_EVENT_ID>,
      ...
    ],
    "blacklisted": [
      <VALIDATOR_EVENT_ID>,
      ...
    ],
    "timeout": <TIMEOUT_IN_MS>,
    "max_qty": <QUANTITY>,
    "languages": {
      <LANGUAGE>: [
          <CAPABILITY>,
          ...
      ],
      ...
    },
    "policy_URI": <POLICY_URI>
  },
  ...
}
```

Their intended meanings are as follows:

- **`whitelisted`:** a list of event IDs for validator events the relay ensures knows about and is willing to execute,
- **`blacklisted`:** a list of event IDs for validator events the relay will never execute,
- **`timeout`:** the number of milliseconds the relay is willing to wait for a _single_ validator execution,
- **`max_qty`:** the maximum number of validators the relay is willing to execute per event,
- **`languages`:** a mapping stating the validator languages the relay is willing to accept; each defines a list of capabilities the relay exposes to the validators execution environment,
- **`policy_uri`:** a URI pointing to the detailed policy text.

Note though that the extremely optional nature of relay validation means that relays are not required to adhere to any of the statements in the presented fields, they're merely informative and not prescriptive in any way.

### 8.2. NIP-20 Command Results

If a relay chooses to validate incoming events and already implements [NIP-20](https://github.com/nostr-protocol/nips/blob/master/20.md), they **SHOULD** respond with the following format when failing validation:

```json
[
  "OK",
  <EVENT_ID>,
  false,
  "invalid: ..."
]
```

Additionally, upon a relay encountering any [Unknown Validators](#71-unknown-validators) listed in the event's tags, it **SHOULD** respond with format:

```json
[
  "OK",
  <EVENT_ID>,
  true,
  "invalid: some unknown validators found [<UNKNOWN_VALIDATOR>, ...]"
]
```

## 9. Client Behavior

Supporting clients **MUST** validate incoming events, although they may do so either _eagerly_ or _lazily_ upon the user's request.

[Invalid Validators](#72-invalid-validators) should be treated as failing validation.
[Unknown Validators](#71-unknown-validators) should be flagged and the user should decide what should be done then.

## 10. Examples

> ---
> TODO
>
> ---

## 11. Use Cases

In what follows, we'll look at some use cases for NOSTR validators.

### 11.1. Oracles

NOSTR can be greatly extended by providing oracles tying an event's validity to the outside world (care must be taken though to prevent the result from one such validation from changing in time, since validation occurs multiple times, it would lead to events being valid or invalid depending on the moment in time in which they occur... on the other hand, ephemeral events could very well use this very property... the mind boggles).

By way of example, we present here a validator that will ensure the Bitcoin network has reached the given block height:

```javascript
// Requires "XMLHttpRequest" capability

const event = arguments[0];                 // get the event being validated
const tagIndex = arguments[1];              // get the tag index
const validatorTag = event.tags[tagIndex];  // extract the validator tag from the event

if (validatorTag[0] !== "v") {  // (OPTIONAL) verify that we are indeed passed a validator tag
  return false;                 // fail if we're not
}

const expectedHeight = validatorTag[2];                                // the expected block height is the next parameter
const url = `https://blockchain.info/block-height/${expectedHeight}`;  // the block-height query URL

const request = new XMLHttpRequest();             // build a new XMLHttpRequest
request.open("GET", url, false);                  // set up a synchronous GET request to the above URL
request.send();                                   // execute it
return request.responseText !== '{"blocks":[]}';  // compare it against the expected error response
```

One would use such a validator with the given validator tag:

```json
[
  "v",
  <VALIDATOR_ID>,
  <EXPECTED_BLOCK_HEIGHT>
]
```

### 11.2. Validator Code Pinning

Validator Code Pinning refers to the act of storing the same validator code in more than one place, and having a validator check that those two places do indeed contain the same code.

This makes it easier to interoperate, since the code itself can be held at a "customary" location (GitHub, GitLab, IPFS, etc.), while the actual validator event lives in the NOSTR network.
A validator performing code pinning would take the "customary" location as an additional parameter, and would be applied to events of kind `1111` (ie. to validators themselves).

One such validator can be very simply implemented:

```javascript
// Requires "XMLHttpRequest" capability

const event = arguments[0];                 // get the event being validated
const tagIndex = arguments[1];              // get the tag index
const validatorTag = event.tags[tagIndex];  // extract the validator tag from the event

if (validatorTag[0] !== "v") {  // (OPTIONAL) verify that we are indeed passed a validator tag
  return false;                 // fail if we're not
}
if (event.kind !== 1111) {      // verify that we are indeed validating a validator
  return false;                 // fail if we're not
}

const canonicalUrl = validatorTag[2];  // the canonical content URL is the next parameter

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

### 11.3. Userland NIP Implementations

Some NIPs can be implemented via validators, this shows that implementing this NIP could transfer protocol maintainability and specialization to the user base, without losing the unicity of specification since validator events are immutable.

#### 11.3.1. NIP-13: Proof of Work

A Proof-of-Work validator can be very simply coded thusly:

```javascript
// Requires "SHA256" capability (hypothetically providing a "SHA256_AS_HEX_STRING()" function)

const event = arguments[0];                 // get the event being validated
const tagIndex = arguments[1];              // get the tag index
const validatorTag = event.tags[tagIndex];  // extract the validator tag from the event

if (validatorTag[0] !== "v") {  // (OPTIONAL) verify that we are indeed passed a validator tag
  return false;                 // fail if we're not
}

// calculate the event ID according to NIP-1
const calculatedId = SHA256_AS_HEX_STRING(
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

const difficulty = event.tags                         // although NIP-13 is unclear as to how to manage
  .filter(tag => tag[0] === "nonce")                  // multiple "nonce" tags, we take the conservative
  .map(tag => tag[2])                                 // approach and consider multiple "nonce" tags as
  .reduce((prev, curr) => prev < curr ? curr : prev)  // describing differing levels of difficulty,
;                                                     // keeping only the highest of them

// this mapping simply stores how many leading 0s a particular hex digit represents
const leading0s = {
  "0": 4,
  "1": 3,
  "2": 2, "3": 2,
  "4": 1, "5": 1, "6": 1, "7": 1,
  "8": 0, "9": 0, "a": 0, "b": 0, "c": 0, "d": 0, "e": 0, "f": 0,
};

var num0s = 0;  // the number of leading 0s

// break the ID into 32-bit blocks and fast-forward the count as long as they are 0
for (let i = 0; i < 32; i += 8) {
  if (parseInt(event.id.substring(i, i + 8), 16)) {
    // deal with the non-0 block and terminate
    for (; i < 32; i++) {
      const currentDigit = event.id[i];
      num0s += leading0s[currentDigit];
      if (currentDigit !== "0") {
        i = 32;  // break from both loops naturally
      }
    }
  } else {
    num0s += 32;
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

## 12. FAQ

**Why use a single-letter tag (ie. `"v"`) for validator tags?**

The reason behind this is twofold:

- on the one hand, this allows for clients to build a `REQ` query restricting themselves to events validated by specific validators, improving efficiency on the client side,
- on the other hand, it allows validators to be freely composed via the `NostrRead` capability: a validator can look for events referring the one being validated that are themselves validated by a specific validators.

Were we not to use a single-letter tag, filtering out the results client-side could be time consuming and cumbersome.

**Why does the validator get passed the tag index?**

This is so so that a validator can "find itself" in the event being validated.
By providing the tag's index, the validator can look into the event's tags for the one being processed, extract the associated event ID as its own (if needed), and pick up the additional arguments therein.

**What's the use of the validator tag `<ADDITIONAL_ARGUMENT>` placeholders?**

This allows a client to construct an event that will forward different parameters to different validators, were we not to have this, we would need to find another place to put these and this would require either encroaching into the `content` field or defining a new field to hold these (or have validators "bake-in" all the required parameters, which would result in an explosion of algorithmically identical validators differing solely on actual arguments passed).

**Can the same validator be specified more than once?**

Indeed, but doing so without varying the additional arguments given in each case would be pointless, and can in fact be optimized away by complying agents.

**Can the validators be run in parallel?**

Absolutely.
Validators **SHOULD** be ["functionally pure"](https://en.wikipedia.org/wiki/Pure_function) in the sense that they should cause no state change whatsoever other than those implied by an event being valid or not.

## 13. Appendixes

## 13.1. Recognized `v-language` Tags

The current NIP recognizes the following languages and their corresponding parameters for usage in the `v-language` tag.

### 13.1.1. JavaScript

The `<LANGUAGE>` placeholder **MUST** be `"javascript"`.

The available `<CAPABILITY>` values are:

- **`XMLHttpRequest`:** makes the `XMLHttpRequest` class available to the validator's code.

Event inputs should be passed as the result of deserializing the JSON value:

```json
[
  <EVENT>,
  <TAG_INDEX>
]
```

and return values interpreted as JavaScript booleans.
Uncaught exceptions are considered `false` return values, except during relay validation, where they're considered `true` return values.

The execution environment will vary depending on the actual type of execution machine (ie. NodeJS vs browser-based), but the execution tactic should be compatible with extracting the value of the following expression:

```javascript
(new Function(validatorEvent.content))(event, tagIndex)
```

where `event` and `tagIndex` are as above, and `validatorEvent` is the validator event referred to by ID in the event's `"v"` tag.

### 13.1.2. Lua

The `<LANGUAGE>` placeholder **MUST** be `"lua"`.

he available `<CAPABILITY>` values are:

> ---
> TODO
>
> ---

Event inputs should be passed as nested Lua tables:

```lua
{
  <EVENT>,
  <TAG_INDEX>
}
```

and return values interpreted as Lua booleans.
Uncaught exceptions are considered `false` return values, except during relay validation, where they're considered `true` return values.
