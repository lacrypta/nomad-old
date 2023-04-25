<!-- markdownlint-enable -->
<!-- markdownlint-disable MD013 -->

# NIP-XX --- Multilingual External Identities in Profiles

> The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**,  **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.txt).

`draft` `optional`

`depends:39`

---

- [1. Motivation](#1-motivation)
- [2. Short Description](#2-short-description)
- [3. Overview](#3-overview)
- [4. Multilingual `"i"` Tag](#4-multilingual-i-tag)
  - [4.1. Predefined Translations](#41-predefined-translations)

---

## 1. Motivation

Te purpose of this NIP is to provide a multilingual alternative to the proofs of [NIP-39](https://github.com/nostr-protocol/nips/blob/master/39.md).

## 2. Short Description

This NIP aims to reduce the hurdles non-English speakers need to enjoy the full feature-set provided by NOSTR.

Specifically, for the implementation of the `"i"` tag validation, users need to commit to a public key via a public message on an external site.
Having an English message appear in an otherwise non-English feed can be jarring, and the purpose of this NIP is to make it less so.

## 3. Overview

This document is organized as follows: first we introduce the new [Multilingual `"i"` Tag](#4-multilingual-i-tag), finally, we provide som [Predefined Translations](#41-predefined-translations) to be used.

## 4. Multilingual `"i"` Tag

A _multilingual `"i"` tag_ is simply a regular `"i"` tag as defined in [NIP-39](https://github.com/nostr-protocol/nips/blob/master/39.md), but instead of consisting solely of the tag itself, a "platform / identity" string, and a proof string, an additional string representing a language **MAY** be added as well at the very end, as an additional tag parameter, we'll call this parameter the `"i"` tag's **_language tag_**.
This new string **MUST** be a valid [RFC 4646](https://www.rfc-editor.org/rfc/rfc4646) language tag (these are the same identifiers used for the HTTP `Accept-Language` headers).

A missing language tag **MUST** be interpreted as being equal to `"en"` (this is stipulated in order to provide backwards compatibility with [NIP-39](https://github.com/nostr-protocol/nips/blob/master/39.md) `"i"` tags).

The presence of a language tag indicates to clients that the expected text to be found in the external site referred to in the proof of a [NIP-39](https://github.com/nostr-protocol/nips/blob/master/39.md) `"i"` should be selected according to its value.
In the next section, predefined translations for some of these language tags are defined.

### 4.1. Predefined Translations

> **NOTE:** this list is non-exhaustive and can be extended in future NIPs.

| Language Tag  | Language              | Expected Text                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ------------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `"en"` / `""` | English               | Verifying that I control the following Nostr public key: `<NPUB ENCODED PUBLIC KEY>`                                                                                                                                                                                                                                                                                                                                                                                               |
| `"ja"`        | Japanese              | &#x6b21;&#x306e; Nostr &#x516c;&#x958b;&#x9375;&#x3092;&#x7ba1;&#x7406;&#x3057;&#x3066;&#x3044;&#x308b;&#x3053;&#x3068;&#x3092;&#x78ba;&#x8a8d;&#x3057;&#x3066;&#x3044;&#x307e;&#x3059;: `<NPUB ENCODED PUBLIC KEY>`                                                                                                                                                                                                                                                               |
| `"es"`        | Spanish               | Verificando que controlo la siguiente clave p&#x00fa;blica de Nostr: `<NPUB ENCODED PUBLIC KEY>`                                                                                                                                                                                                                                                                                                                                                                                   |
| `"de"`        | German                | Best&#x00e4;tigen, dass ich den folgenden &#x00f6;ffentlichen Nostr-Schl&#x00fc;ssel kontrolliere: `<NPUB ENCODED PUBLIC KEY>`                                                                                                                                                                                                                                                                                                                                                     |
| `"ru"`        | Russian               | &#x041f;&#x043e;&#x0434;&#x0442;&#x0432;&#x0435;&#x0440;&#x0436;&#x0434;&#x0435;&#x043d;&#x0438;&#x0435; &#x0442;&#x043e;&#x0433;&#x043e;, &#x0447;&#x0442;&#x043e; &#x044f; &#x043a;&#x043e;&#x043d;&#x0442;&#x0440;&#x043e;&#x043b;&#x0438;&#x0440;&#x0443;&#x044e; &#x0441;&#x043b;&#x0435;&#x0434;&#x0443;&#x044e;&#x0449;&#x0438;&#x0439; &#x043e;&#x0442;&#x043a;&#x0440;&#x044b;&#x0442;&#x044b;&#x0439; &#x043a;&#x043b;&#x044e;&#x0447; Nostr: `<NPUB ENCODED PUBLIC KEY>` |
| `"fr"`        | French                | V&#x00e9;rification que je contr&#x00f4;le la cl&#x00e9; publique Nostr suivante : `<NPUB ENCODED PUBLIC KEY>`                                                                                                                                                                                                                                                                                                                                                                     |
| `"it"`        | Italian               | Verificando di controllare la seguente chiave pubblica Nostr: `<NPUB ENCODED PUBLIC KEY>`                                                                                                                                                                                                                                                                                                                                                                                          |
| `"zh-Hans"`   | Chinese (Simplified)  | &#x9a8c;&#x8bc1;&#x6211;&#x63a7;&#x5236;&#x4ee5;&#x4e0b; Nostr &#x516c;&#x94a5;&#xff1a;`<NPUB ENCODED PUBLIC KEY>`                                                                                                                                                                                                                                                                                                                                                                 |
| `"zh-Hant"`   | Chinese (Traditional) | &#x9a57;&#x8b49;&#x6211;&#x63a7;&#x5236;&#x4ee5;&#x4e0b; Nostr &#x516c;&#x9470;&#xff1a;`<NPUB ENCODED PUBLIC KEY>`                                                                                                                                                                                                                                                                                                                                                                 |
| `"pt"`        | Portuguese            | Verificando se eu controlo a seguinte chave p&#x00fa;blica Nostr: `<NPUB ENCODED PUBLIC KEY>`                                                                                                                                                                                                                                                                                                                                                                                      |
| `"pl"`        | Polish                | Sprawdzam, czy kontroluj&#x0119; nast&#x0119;puj&#x0105;cy klucz publiczny Nostr: `<NPUB ENCODED PUBLIC KEY>`                                                                                                                                                                                                                                                                                                                                                                      |
| `"ar"`        | Arabic                | &#x202B; &#x0627;&#x0644;&#x062a;&#x062d;&#x0642;&#x0642; &#x0645;&#x0646; &#x0623;&#x0646;&#x0646;&#x064a; &#x0623;&#x062a;&#x062d;&#x0643;&#x0645; &#x0641;&#x064a; &#x0645;&#x0641;&#x062a;&#x0627;&#x062d; &#x202A; Nostr &#x202C; &#x0627;&#x0644;&#x0639;&#x0645;&#x0648;&#x0645;&#x064a; &#x0627;&#x0644;&#x062a;&#x0627;&#x0644;&#x064a;: &#x202A; `<NPUB ENCODED PUBLIC KEY>` &#x202C; &#x202C;                                                                           |
| `"fa"`        | Persian               | &#x202B; &#x062a;&#x0623;&#x06cc;&#x06cc;&#x062f; &#x0627;&#x06cc;&#x0646;&#x06a9;&#x0647; &#x0645;&#x0646; &#x06a9;&#x0644;&#x06cc;&#x062f; &#x0639;&#x0645;&#x0648;&#x0645;&#x06cc; &#x202A; Nostr &#x202C; &#x0632;&#x06cc;&#x0631; &#x0631;&#x0627; &#x06a9;&#x0646;&#x062a;&#x0631;&#x0644; &#x0645;&#x06cc; &#x06a9;&#x0646;&#x0645;: &#x202A; `<NPUB ENCODED PUBLIC KEY>` &#x202C; &#x202C;                                                                                 |
| `"id"`        | Indonesian            | Memverifikasi bahwa saya mengontrol kunci publik Nostr berikut: `<NPUB ENCODED PUBLIC KEY>`                                                                                                                                                                                                                                                                                                                                                                                        |
| `"nl"`        | Dutch                 | Verifi&#x00eb;ren dat ik de volgende publieke sleutel van Nostr beheer: `<NPUB ENCODED PUBLIC KEY>`                                                                                                                                                                                                                                                                                                                                                                                |
| `"tr"`        | Turkish               | &#x015e;u Nostr genel anahtar&#x0131;n&#x0131; kontrol etti&#x011f;im do&#x011f;rulan&#x0131;yor: `<NPUB ENCODED PUBLIC KEY>`                                                                                                                                                                                                                                                                                                                                                      |

Taken (quite arbitrarily) from ["Wikipedia page views by language"](https://web.archive.org/web/20230401073217/https://en.wikipedia.org/wiki/Languages_used_on_the_Internet#Wikipedia_page_views_by_language).
Translations were generated by Google Translate, with no oversight whatsoever.
