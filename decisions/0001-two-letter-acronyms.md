---
id: "0001"
title: Two-Letter Acronyms
status: stable
affects: Case Folding
corpus: ["002", "003", "004"]
---

## Question

Should two-letter acronyms (IP, DB, IO) keep both letters capitalized
when rendered in Pascal or camel styles — matching Microsoft's own
naming convention, which capitalizes both letters of a two-letter
acronym but only the first letter of longer ones (`IPAddress` but
`XmlHttpRequest`) — or should they fold under the same one-capital-letter
rule applied to every other acronym regardless of length?

## Decision

No length exception, ever. A two-letter acronym is folded exactly like
a longer one: first letter capitalized, rest lowercased. `IPAddress`
becomes `IpAddress`, not `IPAddress`. `DBConnectionPool` becomes
`DbConnectionPool`. `IOStream` becomes `IoStream`, which also rejects
the .NET convention of treating `IO` as an unsplit unit in
`System.IO`.

## Rationale

A length exception requires the renderer to know, at Serialization
time, that a given token is an acronym rather than an ordinary word —
reintroducing per-token special-casing at exactly the stage this
specification is designed to keep mechanical. Once Tokenization has
produced a token, Case Folding treats all tokens identically; nothing
downstream re-inspects a token's provenance.
