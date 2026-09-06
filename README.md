# commoncase

A specification for converting identifier-like strings into Pascal,
camel, snake, kebab, and constant case, through a deterministic
four-stage pipeline: Normalization, Tokenization, Case Folding,
Serialization.

- [`SPEC.md`](SPEC.md) — the specification.
- [`corpus/cases.json`](corpus/cases.json) — worked examples every
  conforming implementation must match.
- [`decisions/`](decisions) — the decision log behind non-obvious
  rules (see [`0000-template.md`](decisions/0000-template.md) to add
  one).

## Usage

```sh
mise run validate         # check the corpus against the rendering rules
mise run create-decision -- "Title" # scaffold a new decision record
```
