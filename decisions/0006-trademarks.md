---
id: "0006"
title: Trademarks
status: proposed
affects: Tokenization
corpus: []
---

## Question

Some strings split into two linguistically valid words — `git` +
`hub` — but the combined form is a trademark that is conventionally
treated as one atomic identity. Should the Lexicon be allowed to force
such a string to stay one token, even though the mechanical
segmentation isn't wrong?

## Decision

Tentatively, yes: the Lexicon MAY force a string to remain atomic by
policy alone (`github`, not `git` + `hub`).

## Rationale

This differs in kind from [[0004-tokenizer-word-list]]: that decision
uses the Lexicon to fix segmentation that is mechanically broken. This
one would use the same mechanism to override segmentation that is
already correct, purely for stylistic or brand reasons.

## Open question

The current inclusion criterion is effectively "someone noticed it."
That is not a rule, just one person's taste, and needs a sourced
criterion — e.g. a registered-trademark test, or a maintained external
list — before this decision can move from proposed to stable.
