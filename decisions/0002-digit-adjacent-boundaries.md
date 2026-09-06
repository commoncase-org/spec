---
id: "0002"
title: Digit-Adjacent Boundaries
status: stable
affects: Tokenization
corpus: ["007", "008", "009", "010", "011"]
---

## Question

Where do token boundaries fall when a digit sits next to a letter? A
digit can follow a lowercase letter, follow an uppercase letter, or
precede an uppercase or lowercase letter, and each of those
adjacencies could independently be treated as a boundary or a fusion.

## Decision

Three rules govern digit adjacency:

- **T3 — Letter to digit is never a boundary.** The digit fuses onto
  the preceding letters (`sha` + `256` stays `sha256`).
- **T4 — Digit to uppercase letter is always a boundary.** (`256` +
  `Hash` splits.)
- **T5 — Digit to lowercase letter is never a boundary.** This
  sub-case has no corpus case exercising it directly (every digit run
  in the corpus is followed either by a boundary-triggering uppercase
  letter or by nothing) and is therefore unvalidated; see
  [[0007-ordinal-digit-suffix]].

## Rationale

Fusing letters into a following digit (T3) matches how these
identifiers are actually spelled in the wild: `sha256`, `utf8`,
`iso8601`, `aes256`, `top10` are conventionally written as single
units, not as a word awkwardly split before its trailing number.
Treating digit-to-uppercase as a boundary (T4) is what lets
`Sha256Hash` recover the two tokens `sha256` and `hash` instead of
collapsing into one.
