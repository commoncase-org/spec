---
id: "0011"
title: Conformance Profiles and Character Validation
status: stable
affects: Conformance, Normalization, Tokenization
corpus: ["031", "032", "033", "034"]
---

## Question

How should commoncase validate input characters, handle
non-alphanumeric ASCII symbols and punctuation (e.g., `@`, `#`, `+`,
`&`), and reconcile the competing requirements of strict
compiler/code-generation toolchains versus permissive text
slugification?

## Decision

This specification establishes two normative Conformance Profiles
orthogonal to serialization Conformance Classes:

1. **Strict Profile (Strict commoncase):**
   - The character repertoire following Normalization MUST consist
     strictly of alphanumeric characters (`ALPHANUM`: `[A-Za-z0-9]`)
     and explicit delimiters (`DELIM`: `-`, `_`, space, `.`, `/`).
   - If an input string contains any character outside this
     repertoire (including ASCII symbols like `@`, `#`, `$`, `%`, `&`,
     `+`, `=`, control characters, or unmapped Unicode code points),
     the processor MUST terminate execution immediately and reject the
     input with an `InvalidCharacterError`.
   - Core commoncase MUST NOT perform heuristic or hardcoded symbol
     transliterations (such as replacing `@` with `"at"` or `&` with
     `"and"`). Any such verbalization or Unicode CLDR-based
     transformation MUST be executed as an upstream pre-processing step
     prior to invoking Strict commoncase.

2. **Basic Profile (Basic commoncase):**
   - Extends delimiter segmentation under Rule D1-B: any contiguous
     sequence of characters outside `ALPHANUM` (including all ASCII
     printable punctuation, symbols, ASCII whitespace, and unmapped
     non-ASCII characters) MUST be treated as a delimiter boundary.
     All delimiter characters are discarded and omitted from tokens.
     Leading and trailing delimiters are stripped.
   - If an input contains no alphanumeric characters, it evaluates to
     an empty token sequence and serializes to the empty string `""`.

3. **Security Invariants Apply Universally:**
   - Both profiles MUST strictly enforce the security bounds
     established in ADR 0010: maximum input length limit ($\le 256$
     scalars) rejected with `PayloadTooLargeError`, rejection of Trojan
     Source and Unicode bidirectional control characters with
     `SecurityValidationError`, and linear $O(N)$ scanning complexity.

## Rationale

Code generators, AST transpilers, and ORM deserializers require
uncompromising determinism and fail-fast guarantees. If a compiler
toolchain transforms `user@domain` or `c++` by silently dropping
symbols, it risks silent identifier collisions, mass-assignment
security vulnerabilities, and broken code.

Conversely, search query parsers, CMS slugifiers, and CLI utilities
frequently encounter raw human input containing commas, exclamation
marks, or currency symbols. Raising fatal exceptions on every symbol
makes integration hostile for permissive domains.

Baking symbol verbalization (`@` $\to$ `"at"`, `&` $\to$ `"and"`)
into core commoncase is linguistically biased and semantically flawed:
in German `@` is `At-Zeichen` or `bei`, `+` in `+1-800` is a phone
prefix that should be stripped rather than expanded, and in `$foo` the
dollar sign is a variable sigil. Separating symbol verbalization into
an upstream pre-processing step preserves core commoncase's strict
locale invariance while enabling applications to utilize Unicode CLDR
annotations or domain-specific rules where appropriate.

## Examples

### Positive (Strict Profile)

- `user_id` → tokens: `user`, `id` (valid DELIM `_`)
- `System.IO` → tokens: `system`, `io` (valid DELIM `.`)
- `path/to/file` → tokens: `path`, `to`, `file` (valid DELIM `/`)

### Negative (Strict Profile)

- `user@domain.com` → Rejected with `InvalidCharacterError`
  (character `@` at index 4).
- `c++` → Rejected with `InvalidCharacterError`
  (character `+` at index 1).
- `50%Off` → Rejected with `InvalidCharacterError`
  (character `%` at index 2).
- `hello, world!` → Rejected with `InvalidCharacterError`
  (character `,` at index 5).

### Positive (Basic Profile)

- `user@domain.com` → tokens: `user`, `domain`, `com`
  - camel: `userDomainCom`
  - snake: `user_domain_com`
- `c++` → tokens: `c` (trailing `++` stripped as delimiters)
  - Pascal: `C`
- `50%Off` → tokens: `50`, `off`
  - snake: `50_off`
- `Price ($USD)` → tokens: `price`, `usd`
  - camel: `priceUsd`
- `hello, world!` → tokens: `hello`, `world`
  - snake: `hello_world`
- `$$$` → tokens: `[]` → Pascal: `""`, snake: `""`

### Upstream Pre-Processing Flow

- Raw application input: `"C++ Developer & Designer"`
- Upstream pre-processing (CLDR / domain lexicon):
  `"cpp developer and designer"`
- Invoking Strict commoncase:
  - snake: `cpp_developer_and_designer`
  - Pascal: `CppDeveloperAndDesigner`
