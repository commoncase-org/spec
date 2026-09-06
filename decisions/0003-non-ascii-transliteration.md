---
id: "0003"
title: Non-ASCII Transliteration
status: stable
affects: Normalization
corpus: ["021", "022"]
---

## Question

How should input containing umlauts and eszett (ä, ö, ü, ß) be handled
before Tokenization runs? Left as-is, stripped, or replaced with an
ASCII-only equivalent?

## Decision

Transliterate at Normalization, before any other stage runs: ä → ae,
ö → oe, ü → ue, ß → ss.

## Rationale

Keeps every later stage — Tokenization, Case Folding, Serialization —
ASCII-only, so none of them need to reason about non-ASCII input or
locale-specific casing rules. The scope of this decision is
German-only; other scripts (accented Romance-language letters,
non-Latin scripts, etc.) are unaddressed and left for a future
decision record.
