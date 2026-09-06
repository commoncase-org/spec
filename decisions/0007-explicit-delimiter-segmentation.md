---
id: "0007"
title: Explicit Delimiter Segmentation
status: stable
affects: Tokenization
corpus: ["023", "024", "025", "026"]
---

## Question

How should explicit delimiter characters (hyphen `-`, underscore `_`,
space ` `, dot `.`, slash `/`) in input strings be processed during
Tokenization?

## Decision

Explicit delimiter characters denote hard boundaries under Rule D1.
Any contiguous sequence of one or more delimiter characters MUST be
treated as a boundary. Delimiter characters MUST be discarded and
MUST NOT be emitted into any token. Leading and trailing delimiters
MUST be stripped without creating empty tokens. Consecutive
delimiters MUST be collapsed into a single boundary.

## Rationale

The primary purpose of an identifier case transformation engine is
converting between conventions, including snake_case and kebab-case.
Without an explicit delimiter rule, input strings containing
underscores or hyphens lack normative segmentation semantics. Treating
delimiters as boundaries that are purged upon tokenization ensures
that inputs such as `user_id`, `user-id`, and `userId` all produce the
identical canonical token sequence `["user", "id"]`.

## Examples

### Positive

- `user_id` → tokens: `user`, `id`
  - Pascal: `UserId`
  - camel: `userId`
  - kebab: `user-id`
- `user-id` → tokens: `user`, `id`
  - snake: `user_id`
- `__init__` → tokens: `init`
  - Pascal: `Init`
  (Leading and trailing delimiters stripped; consecutive delimiters
  collapsed.)
- `path/to/file` → tokens: `path`, `to`, `file`
  - snake: `path_to_file`

### Negative

- `user_id` → tokens: `user_id`
  (Rejected: treating delimiters as delimiter-free runs.)
- `__init__` → tokens: `""`, `""`, `init`, `""`, `""`
  (Rejected: emitting empty tokens for consecutive or boundary
  delimiters.)
