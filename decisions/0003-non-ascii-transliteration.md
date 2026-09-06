---
id: "0003"
title: Non-ASCII Transliteration
status: stable
affects: Normalization
corpus: ["021", "022"]
---

## Question

How should non-ASCII input be handled before Tokenization runs? Left
as-is, stripped, or replaced with an ASCII-only equivalent?

## Decision

Transliterate at Normalization, before any other stage runs, using a
mapping table rather than a per-script special case. The table
currently defines the German characters that motivated this decision:
ä → ae, ö → oe, ü → ue, ß → ss.

## Rationale

Keeps every later stage — Tokenization, Case Folding, Serialization —
ASCII-only, so none of them need to reason about non-ASCII input or
locale-specific casing rules. The mechanism itself is general (any
non-ASCII character can get a mapping table entry); only the table's
current contents are German-specific. Other scripts (accented
Romance-language letters, non-Latin scripts, etc.) have no entries yet
and remain unaddressed until a concrete case motivates adding one.
