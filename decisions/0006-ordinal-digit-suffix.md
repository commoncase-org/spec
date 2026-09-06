---
id: "0006"
title: Ordinal Digit Suffix
status: stable
affects: Tokenization
corpus: []
---

## Question

Should transitions from a digit to a lowercase letter (such as
ordinal suffixes in `2ndChance`) be treated as a boundary, fused
uniformly under Rule T5, or parsed using specialized English-language
ordinal detection?

## Decision

Digit-to-lowercase transitions are never boundaries (Rule T5). A
lowercase letter immediately following a digit fuses onto the
preceding digit run. No special-case parser or dictionary lookup is
introduced for English ordinals (`st`, `nd`, `rd`, `th`); all
digit-to-lowercase sequences follow the identical rule. Any domain
exception requiring a split must be declared explicitly via the
Lexicon.

## Rationale

Special-casing ordinals would inject English-specific linguistic
heuristics into an otherwise language-agnostic, orthographic pipeline.
Treating digit-to-lowercase as non-boundaries maintains direct
symmetry with Rule T3 (letter-to-digit fusion) and avoids heuristic
guessing. Any unintended fusions in delimiter-free sequences can be
disambiguated deterministically via the Lexicon mechanism established
in [[0004-tokenizer-word-list]] and [[0005-ambiguous-compound-words]].

## Examples

### Positive

- `2ndChance` → tokens: `2nd`, `chance`
  - Pascal: `2ndChance`
  - camel: `2ndChance`
  - snake: `2nd_chance`
  (Ordinal suffix `nd` fuses onto preceding digit `2` under Rule T5,
  and splits before uppercase `C` under Rule T4.)
- `win32api` → tokens: `win32api`
  - snake: `win32api`
  (Digit `32` fuses onto preceding `win` under Rule T3, and `api`
  fuses onto `32` under Rule T5.)

### Negative

- `2ndChance` → tokens: `2`, `nd`, `chance`
  (Rejected: splitting a digit run from its ordinal suffix.)
- Hardcoding specialized English-language ordinal detection into the
  mechanical tokenizer.
