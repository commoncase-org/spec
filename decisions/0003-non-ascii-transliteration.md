---
id: "0003"
title: Non-ASCII Transliteration
status: stable
affects: Normalization
corpus: ["021", "022"]
---

## Question

How should non-ASCII input be handled before Tokenization runs, and
against what external convention, if any, should the mapping be
sourced?

## Decision

Transliterate at Normalization, before any other stage runs, using a
closed, deterministic mapping table sourced from ICAO Doc 9303
(*Machine Readable Travel Documents*, Part 3, Section 6).

## Rationale

Keeps every later stage — Tokenization, Case Folding, Serialization —
ASCII-only, so none of them need to reason about non-ASCII input or
locale-specific casing rules.

ICAO Doc 9303 provides a closed, deterministic, per-character mapping
table that avoids algorithmic or locale-dependent heuristics. It
solves the requirement for canonical, collision-aware ASCII
transliteration per identifier, while defining authoritative mappings
for multiple languages beyond German. Characters without an ICAO 9303
transliteration remain unaddressed until a concrete use case
motivates adding one.

## Examples

### Positive

- `müllerStraße` → Normalized: `muellerStrasse`
  - tokens: `mueller`, `strasse`
  - Pascal: `MuellerStrasse`
  - snake: `mueller_strasse`
- `straße` → Normalized: `strasse`
  - tokens: `strasse`
  - constant: `STRASSE`

### Negative

- `müllerStraße` → `mullerStrasse`
  (Rejected: stripping diacritics instead of transliterating per
  ICAO 9303.)
- `müllerStraße` → `müller_strasse`
  (Rejected: passing non-ASCII characters beyond Normalization.)
