---
id: "0005"
title: Ambiguous Compound Words
status: stable
affects: Tokenization
corpus: ["017", "018", "019"]
---

## Question

Delimiter-free runs like `username`, `filepath`, and `metadata` carry
no orthographic signal at all — no case change, no digit, nothing for
the Orthographic Segmentation Rules to act on. Should Tokenization
attempt dictionary-based segmentation to guess at sub-words anyway?

## Decision

No heuristic segmentation. The default for a delimiter-free run is one
token. Specific Lexicon entries may override that default in either
direction: forcing a split (`filepath` → `file` + `path`) or
explicitly confirming that none should happen (`username`, `metadata`
stay whole).

## Rationale

Heuristic longest-match dictionary segmentation risks confidently wrong
splits with no way to detect the error at the point it's made. A
closed Lexicon list, by contrast, is wrong only by omission — an
unlisted string simply falls back to the one-token default — never by
false confidence in a bad guess.
