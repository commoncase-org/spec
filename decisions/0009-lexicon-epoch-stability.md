---
id: "0009"
title: Lexicon Epoch Stability
status: stable
affects: Tokenization
corpus: ["012", "013", "014", "017"]
---

## Question

How can the Lexicon evolve over time without silently mutating existing
identifiers and breaking downstream database schemas, ORM models, and
public API contracts?

## Decision

The Lexicon is governed by an immutable Epoch stability model:

1. **Immutability of Standard Lexicon:** The Standard Core Lexicon
   defined in Annex C is an immutable versioned dictionary. Once a
   specification version is ratified, entries in the Standard Core
   Lexicon MUST NOT be added, modified, or removed in subsequent minor
   or patch releases.
2. **Major Version Bump for Core Lexicon Mutations:** Any modification
   or addition to the Standard Core Lexicon constitutes a breaking
   change and MUST trigger a MAJOR version increment under Semantic
   Versioning (SemVer 2.0.0).
3. **User-Defined Lexicon Extensions:** Implementations MAY accept
   user-defined domain lexicons via an explicit runtime configuration
   interface. User-defined lexicons take precedence over the Standard
   Core Lexicon but MUST NOT alter the default output of conforming
   implementations in their standard configuration.

## Rationale

In persistent software systems (relational databases, ORMs, serialized
network payloads, protobuf definitions), identifier casing is not
ephemeral display formatting—it defines schema contracts. If an
additive release of `commoncase` in a minor version split `filepath`
into `file_path`, automated ORM migrations would drop existing columns,
and API serializers would break backward compatibility with legacy
clients. Isolating Lexicon updates to immutable epochs ensures that
upgrades remain safe and deterministic.

## Examples

### Positive

- Upgrading from `v1.0.0` to `v1.1.0` guarantees that every identifier
  in existence produces bit-for-bit identical tokens and serialization
  under the Standard Core Lexicon.
- Applications requiring domain-specific acronyms (`k8s`, `graphql`)
  register an explicit user-defined lexicon profile without mutating
  the global standard baseline.

### Negative

- Adding a newly popularized technology acronym to the default Lexicon
  in a minor release `v1.1.0`, causing previously generated database
  columns to be renamed.
- Modifying token decompositions of existing Lexicon terms across minor
  versions.
