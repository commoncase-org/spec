---
id: "0010"
title: Security Considerations and Complexity Bounds
status: stable
affects: Pipeline
corpus: []
---

## Question

What security guarantees and computational complexity bounds must be
normatively enforced to prevent canonicalization collisions, Denial of
Service (ReDoS / memory exhaustion), and Trojan Source injection?

## Decision

The pipeline is bound by four normative security invariants:

1. **Input Length Bounding:** Implementations MUST enforce a maximum
   input length limit. The default maximum limit MUST NOT exceed 256
   Unicode scalar values (or UTF-8 bytes). Inputs exceeding this limit
   MUST be rejected immediately prior to Normalization.
2. **Linear Time Execution ($O(N)$):** All pipeline stages MUST
   execute in $O(N)$ linear time relative to input length. Tokenization
   MUST NOT rely on backtracking regular expressions capable of
   catastrophic backtracking (ReDoS). Tokenization MUST be implemented
   via deterministic finite automata (DFAs) or single-pass linear
   character scanners.
3. **Rejection of Trojan Source & Invisible Characters:** Inputs
   containing Unicode bidirectional formatting controls (e.g., `U+202A`
   through `U+202E`, `U+2066` through `U+2069`) or default-ignorable
   characters MUST be rejected with a `SecurityValidationError`.
4. **Locale-Invariant Casing:** Case Folding and Serialization MUST
   operate on ASCII bitwise or table mappings independent of host
   environment locale settings (specifically avoiding the Turkish
   dotless-i casing anomaly).

## Rationale

Because `commoncase` is targeted at compilers, code generators, and
public API deserializers, untrusted inputs can be weaponized against
the host environment. A payload with 50 uppercase letters followed by
punctuation can trigger seconds of CPU starvation in naive regex
tokenizers. Large payloads of alternating case can trigger gigabytes of
heap allocations, crashing containerized processes. Invariant ASCII
casing ensures that identical inputs produce identical output regardless
of whether execution occurs in an English or Turkish locale.

## Examples

### Positive

- A 100-character identifier `getUserByAuthenticationToken` evaluates
  in strict linear time using single-pass state transitions.
- Running constant serialization on `userId` produces `USER_ID`
  uniformly across all operating system locales.

### Negative

- Evaluating Cap-run Rule T2 using nested backtracking regexes that
  hang the processor on `"A" * 50 + "!"`.
- Accepting inputs with embedded Right-to-Left Override (`U+202E`)
  characters that invert visual display in IDEs and code review tools.
