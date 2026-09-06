# commoncase

A specification for converting identifier-like strings into Pascal,
camel, snake, kebab, and constant case, through a deterministic
four-stage pipeline: Normalization, Tokenization, Case Folding,
Serialization.

- [`SPEC.md`](SPEC.md) — compiled normative specification.
- [`spec/SPEC.template.md`](spec/SPEC.template.md) — source template.
- [`data/`](data) — normative registries (transliterations, lexicon).
- [`conformance/vectors.json`](conformance/vectors.json) — golden test
  vectors every conforming implementation must match.
- [`decisions/`](decisions) — the decision log behind non-obvious
  rules (see [`0000-template.md`](decisions/0000-template.md) to add
  one).

## Usage

```sh
mise run build            # build SPEC.md from template using Knap
mise run validate         # validate conformance vectors & registries
mise run create-decision -- "Title" # scaffold a new decision record
```
