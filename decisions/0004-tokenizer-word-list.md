---
id: "0004"
title: Tokenizer Word List
status: stable
affects: Tokenization
corpus: ["012", "013", "014"]
---

## Question

For some strings, mechanical segmentation is flatly wrong: `oauth` and
`graphql` do not decompose into meaningful sub-words the way `sha256`
or `http` do. How can this be fixed without introducing a render-time
exception?

## Decision

A curated, additive Lexicon may override Tokenization's output for a
specific string, before Case Folding runs. This is the only point in
the entire pipeline where domain knowledge is injected.

## Rationale

Applying the ordinary Orthographic Segmentation Rules to `OAuth2Client`
mechanically splits it as `o` + `auth2` + `client`, which Case Folding
and Serialization would then reassemble right back into the rejected
spelling `OAuth2Client` — the exact string the rule was meant to fix.
An opaque, preserve-verbatim Serialization step was already rejected
elsewhere in this specification, so the correction has to happen
upstream, in Tokenization, before the bad segmentation ever reaches
Case Folding.

## Examples

### Positive

- `OAuth2Client` → tokens: `oauth2`, `client`
  - Pascal: `Oauth2Client`
  - camel: `oauth2Client`
  - snake: `oauth2_client`
  (Lexicon overrides mechanical segmentation before Case Folding.)
- `GraphQLSchema` → tokens: `graphql`, `schema`
  - Pascal: `GraphqlSchema`

### Negative

- `OAuth2Client` → tokens: `o`, `auth2`, `client`
  - Pascal: `OAuth2Client`
  (Rejected: mechanical segmentation breaking the acronym into `o` and
  `auth2`.)
