---
id: "0008"
title: Unicode NFC and Normative Transliteration Table
status: stable
affects: Normalization
corpus: ["021", "022", "029", "030"]
---

## Question

How should non-ASCII input strings be normalized and transliterated
into ASCII before Tokenization, and how should canonical equivalence
and unmapped code points be handled?

## Decision

Normalization operates in two consecutive, deterministic steps:

1. **Canonical Pre-Normalization:** The input string MUST be
   normalized to Unicode Normalization Form C (NFC) per [UAX15] prior
   to character transliteration.
2. **Deterministic Table Lookup:** Code points matching an entry in
   the Normative Transliteration Table (Annex B /
   `annex/transliteration.csv`) MUST be replaced with their designated
   ASCII replacement sequence. Both lowercase and uppercase variants
   are explicitly mapped.
3. **Fail-Fast Error Model:** If an input string contains any
   non-ASCII code point not present in the Normative Transliteration
   Table, the processor MUST reject the input with an
   `UnsupportedCharacterError`. Silent deletion, substitution of
   replacement characters (e.g., `?` or `U+FFFD`), and un-normalized
   passthrough are strictly prohibited.

## Rationale

Without NFC pre-normalization, canonically equivalent Unicode strings
diverge based on operating system conventions. On macOS, input
subsystems and filesystems emit decomposed NFD strings (e.g., `u` +
combining diaeresis `U+0308`), whereas Linux and Windows emit
precomposed NFC strings (`ü` / `U+00FC`). Applying NFC first
guarantees that decomposed sequences collapse into precomposed code
points before table lookup.

Embedding the transliteration table directly in the specification
eliminates external dependencies and removes the ambiguities present
in raw ICAO Doc 9303 text (such as "AE or A" options or missing
lowercase mappings). An explicit fail-fast error model ensures that
unsupported scripts or emojis do not cause silent data corruption or
security filter bypasses.

## Examples

### Positive

- `müllerStraße` (precomposed NFC) → Normalized: `muellerStrasse`
  - tokens: `mueller`, `strasse`
  - Pascal: `MuellerStrasse`
  - snake: `mueller_strasse`
- `mu\u0308ller` (decomposed NFD) → Normalized: `mueller`
  - Pascal: `Mueller`
  (NFC pre-normalization composes `u` + diaeresis before lookup.)
- `Äpfel` → Normalized: `Aepfel`
  - tokens: `aepfel`
  - camel: `aepfel`
  - constant: `AEPFEL`

### Negative

- `mu\u0308ller` → `muller`
  (Rejected: failing to pre-normalize to NFC, resulting in loss of the
  combining diaeresis.)
- `user🚀name` → `username`
  (Rejected: silently stripping unmapped emoji instead of raising an
  error.)
- `müller` → `muller`
  (Rejected: diacritic stripping instead of phonetic transliteration.)
