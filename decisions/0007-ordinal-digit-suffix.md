---
id: "0007"
title: Ordinal Digit Suffix
status: proposed
affects: Tokenization
corpus: []
---

## Question

Does the digit-to-lowercase default set by T5 in
[[0002-digit-adjacent-boundaries]] — never a boundary — hold generally,
or only for the ordinal case that motivated it (`2nd` fusing rather
than splitting as `2` + `nd`)?

## Decision

None yet. This record is a placeholder marking the open question until
corpus cases exist to test candidate answers.

## Open question

No corpus case currently exercises a digit immediately followed by a
lowercase letter, so T5 is asserted by symmetry with T3 rather than by
evidence. Whether that symmetry actually holds — or whether ordinals
are a special case distinct from the general digit-to-lowercase
adjacency — is unresolved.
