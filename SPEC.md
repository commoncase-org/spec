# commoncase Specification

## Preface

commoncase is a specification for converting a single identifier-like
input string into one of five case styles - Pascal, camel, snake,
kebab, and constant - through a documented, deterministic pipeline.
The pipeline has exactly four stages, always run in the same order:
Normalization, Tokenization, Case Folding, and Serialization. Every
normative rule in this specification is paired with at least one
example drawn from the project's corpus (`corpus/cases.json`), and
every rule that required a judgment call is backed by a decision
record under `decisions/` (indexed in Annex A). Where a rule has no
decision record, its corpus cases carry a `decision` value of `null`,
meaning the rule is considered uncontested.

## References

### Normative References

- RFC 2119, *Key words for use in RFCs to Indicate Requirement
  Levels*. Defines **MUST**, **MUST NOT**, **SHOULD**, and **MAY** as used throughout
  this specification.

### Informative References

- Microsoft's .NET Framework naming conventions, which capitalize both
  letters of a two-letter acronym (e.g. `IPAddress`). Cited in
  `decisions/0001-two-letter-acronyms.md` as the convention this
  specification decides against.
- The .NET `System.IO` namespace, which treats `IO` as an unsplit
  unit. Cited in `decisions/0001-two-letter-acronyms.md` as a second
  convention this specification decides against.
- Google's identifier-naming guidance, whose canonical example
  `XMLHttpRequest` is used in `decisions/0001-two-letter-acronyms.md`
  as a positive precedent this specification follows.

## Terminology

The following terms are used throughout this specification with the
meanings given here. No other term is used as a synonym for any of
these.

**Token** - the smallest unit of meaning that Tokenization identifies
within an input string. Case Folding and Serialization operate only on
Tokens; neither stage ever inspects the original input string.

**Boundary** - a position between two characters of an input string at
which Tokenization divides the string into two separate Tokens.

**Orthographic signal** - a feature of an input string's spelling - a
change in letter case, the presence of a digit, or an explicit
delimiter character - that Tokenization examines to decide whether a
Boundary exists at a given position.

**Delimiter-free run** - a maximal substring that contains no
delimiter character and no other Orthographic signal capable of
marking a Boundary. Tokenization treats a Delimiter-free run as a
single Token unless the Lexicon specifies otherwise.

**Cap-run** - a maximal run of two or more consecutive uppercase
letters within an input string.

**Lexicon** - the curated, additive list that may override
Tokenization's mechanical output for a specific string, applied before
Case Folding runs.

**Decision record** - a document under `decisions/` that records a
Question, a Decision, and a Rationale (and, where the matter is
unresolved, an Open question) governing one or more rules in this
specification, together with the corpus case IDs it governs.

## Scope

This specification defines a pipeline for converting a single
identifier-like input string - such as a variable, class, or field
name - into five output styles: Pascal, camel, snake, kebab, and
constant. It defines the four pipeline stages (Normalization,
Tokenization, Case Folding, Serialization), the rules applied at each
stage, and the mechanism (the Lexicon) by which curated exceptions to
the mechanical rules are recorded.

This specification does not address: segmentation of natural-language
prose or multi-identifier strings; transliteration of non-ASCII
characters beyond the mapping table currently defined in Normalization
(see `decisions/0003-non-ascii-transliteration.md`); or a settled
policy for treating trademarked compounds as atomic (see
`decisions/0006-trademarks.md`, which remains proposed).

## Conformance

An implementation of this specification conforms if and only if it:

1. performs Normalization, then Tokenization, then Case Folding, then
   Serialization, in that order, with no stage skipped or reordered;
   and
2. produces output matching every case in `corpus/cases.json` for at
   least one of the five Serialization styles.

```example
  input: userId
  tokens: user, id
  pascal: UserId
  camel: userId
  snake: user_id
  kebab: user-id
  constant: USER_ID
```

## Normalization

Normalization **MUST** transliterate non-ASCII characters to ASCII before
Tokenization runs. The mapping table currently defines: a with an
umlaut to "ae", o with an umlaut to "oe", u with an umlaut to "ue",
and the eszett character to "ss". After Normalization, the string
**MUST** contain only ASCII characters. (See
`decisions/0003-non-ascii-transliteration.md`.)

```example
  input: straße
  tokens: strasse
  pascal: Strasse
  camel: strasse
  snake: strasse
  kebab: strasse
  constant: STRASSE
```

```example
  input: müllerStraße
  tokens: mueller, strasse
  pascal: MuellerStrasse
  camel: muellerStrasse
  snake: mueller_strasse
  kebab: mueller-strasse
  constant: MUELLER_STRASSE
```

## Tokenization

Tokenization **MUST** divide a Normalized input string into a sequence of
Tokens. It does this in two steps: first, the Orthographic Segmentation
Rules mechanically identify Boundaries; second, the Lexicon **MAY**
override the mechanical result for specific strings before Case
Folding runs.

### Orthographic Segmentation Rules

The following rules, labeled T1 through T5 with one lettered sub-rule,
govern where Tokenization places a Boundary based on adjacent
characters.

**T1** - A transition from a lowercase letter to an uppercase letter
**MUST** be treated as a Boundary.

```example
  input: userId
  tokens: user, id
  pascal: UserId
  camel: userId
  snake: user_id
  kebab: user-id
  constant: USER_ID
```

**T2** - A Cap-run followed by a lowercase letter **MUST** have a Boundary
placed immediately before the Cap-run's final letter, so that the
final letter joins the following lowercase run instead of the Cap-run.

```example
  input: IPAddress
  tokens: ip, address
  pascal: IpAddress
  camel: ipAddress
  snake: ip_address
  kebab: ip-address
  constant: IP_ADDRESS
```

**T2a** - A single uppercase letter does not constitute a Cap-run (T2
requires two or more consecutive uppercase letters); a lone uppercase
letter preceded by a lowercase letter is governed by T1 alone.

```example
  input: iPhone
  tokens: i, phone
  pascal: IPhone
  camel: iPhone
  snake: i_phone
  kebab: i-phone
  constant: I_PHONE
```

```example
  input: xCoordinate
  tokens: x, coordinate
  pascal: XCoordinate
  camel: xCoordinate
  snake: x_coordinate
  kebab: x-coordinate
  constant: X_COORDINATE
```

**T3** - A transition from a letter to a digit **MUST NOT** be treated as a
Boundary; the digit fuses onto the preceding letters. (See
`decisions/0002-digit-adjacent-boundaries.md`.)

```example
  input: Sha256Hash
  tokens: sha256, hash
  pascal: Sha256Hash
  camel: sha256Hash
  snake: sha256_hash
  kebab: sha256-hash
  constant: SHA256_HASH
```

**T4** - A transition from a digit to an uppercase letter **MUST** be
treated as a Boundary. (See
`decisions/0002-digit-adjacent-boundaries.md`.)

```example
  input: Iso8601Date
  tokens: iso8601, date
  pascal: Iso8601Date
  camel: iso8601Date
  snake: iso8601_date
  kebab: iso8601-date
  constant: ISO8601_DATE
```

**T5** - A transition from a digit to a lowercase letter **MUST NOT** be
treated as a Boundary. This sub-case is unvalidated: no corpus case
exercises a digit immediately followed by a lowercase letter, and it
is asserted only by symmetry with T3. (See
`decisions/0007-ordinal-digit-suffix.md`, which remains proposed.)

### Default Segmentation Behavior

A Delimiter-free run **MUST** default to a single Token when it contains
no Orthographic signal for the Orthographic Segmentation Rules to act
on. This default **MUST NOT** be overridden by heuristic dictionary
segmentation; it **MAY** be overridden only by an explicit Lexicon entry.
(See `decisions/0005-ambiguous-compound-words.md`.)

```example
  input: username
  tokens: username
  pascal: Username
  camel: username
  snake: username
  kebab: username
  constant: USERNAME
```

```example
  input: metadata
  tokens: metadata
  pascal: Metadata
  camel: metadata
  snake: metadata
  kebab: metadata
  constant: METADATA
```

### The Lexicon

The Lexicon is a curated, additive list of exceptions applied after
the Orthographic Segmentation Rules and before Case Folding. Each
Lexicon entry replaces the Tokens that mechanical segmentation would
otherwise produce for a specific string with a corrected set of
Tokens. It exists for three distinct reasons:

- to correct segmentation that the Orthographic Segmentation Rules get
  flatly wrong, such as `oauth2` or `graphql`, which do not decompose
  into their intended sub-words mechanically (see
  `decisions/0004-tokenizer-word-list.md`);
- to force or confirm a split within a Delimiter-free run, where the
  default of one Token is not the intended reading (see
  `decisions/0005-ambiguous-compound-words.md`); and
- to force a mechanically correct split to remain one Token for
  trademark reasons (see `decisions/0006-trademarks.md`, which remains
  **proposed** and is not yet a settled part of this specification).

```example
  input: OAuth2Client
  tokens: oauth2, client
  pascal: Oauth2Client
  camel: oauth2Client
  snake: oauth2_client
  kebab: oauth2-client
  constant: OAUTH2_CLIENT
```

```example
  input: IPv6Address
  tokens: ipv6, address
  pascal: Ipv6Address
  camel: ipv6Address
  snake: ipv6_address
  kebab: ipv6-address
  constant: IPV6_ADDRESS
```

## Case Folding

Case Folding **MUST** capitalize the first letter of a Token and lowercase
its remaining letters, for every style except constant. This rule
applies uniformly to every Token regardless of length or origin - no
exception is made for a Token that happens to be an acronym, and none
is made for a Token supplied by the Lexicon rather than by the
Orthographic Segmentation Rules. (See
`decisions/0001-two-letter-acronyms.md`.)

```example
  input: IPAddress
  tokens: ip, address
  pascal: IpAddress
  camel: ipAddress
  snake: ip_address
  kebab: ip-address
  constant: IP_ADDRESS
```

```example
  input: DBConnectionPool
  tokens: db, connection, pool
  pascal: DbConnectionPool
  camel: dbConnectionPool
  snake: db_connection_pool
  kebab: db-connection-pool
  constant: DB_CONNECTION_POOL
```

## Serialization

Serialization **MUST** produce each of the following five styles from a
Token sequence, using the case forms Case Folding has already applied:

**Pascal** - concatenate every Token with no delimiter, capitalizing
the first letter of each Token.

```example
  input: userId
  tokens: user, id
  pascal: UserId
  camel: userId
  snake: user_id
  kebab: user-id
  constant: USER_ID
```

**camel** - concatenate every Token with no delimiter, identically to
Pascal, except the first Token's leading letter **MUST** remain lowercase.

```example
  input: IPAddress
  tokens: ip, address
  pascal: IpAddress
  camel: ipAddress
  snake: ip_address
  kebab: ip-address
  constant: IP_ADDRESS
```

**snake** - join every Token with a single underscore, with every
Token lowercase.

```example
  input: HTTPRequestHandler
  tokens: http, request, handler
  pascal: HttpRequestHandler
  camel: httpRequestHandler
  snake: http_request_handler
  kebab: http-request-handler
  constant: HTTP_REQUEST_HANDLER
```

**kebab** - join every Token with a single hyphen, with every Token
lowercase.

```example
  input: XMLHttpRequest
  tokens: xml, http, request
  pascal: XmlHttpRequest
  camel: xmlHttpRequest
  snake: xml_http_request
  kebab: xml-http-request
  constant: XML_HTTP_REQUEST
```

**constant** - uppercase every Token and join with a single
underscore.

```example
  input: Aes256Key
  tokens: aes256, key
  pascal: Aes256Key
  camel: aes256Key
  snake: aes256_key
  kebab: aes256-key
  constant: AES256_KEY
```

## Annex A (informative) Decision Records

This annex is informative. It indexes every decision record under
`decisions/`; it does not reproduce their content.

| # | Title | Status | Corpus |
|---|---|---|---|
| [0001](decisions/0001-two-letter-acronyms.md) | Two-Letter Acronyms | stable | 002, 003, 004 |
| [0002](decisions/0002-digit-adjacent-boundaries.md) | Digit-Adjacent Boundaries | stable | 007, 008, 009, 010, 011 |
| [0003](decisions/0003-non-ascii-transliteration.md) | Non-ASCII Transliteration | stable | 021, 022 |
| [0004](decisions/0004-tokenizer-word-list.md) | Tokenizer Word List | stable | 012, 013, 014 |
| [0005](decisions/0005-ambiguous-compound-words.md) | Ambiguous Compound Words | stable | 017, 018, 019 |
| [0006](decisions/0006-trademarks.md) | Trademarks | **proposed** | none yet |
| [0007](decisions/0007-ordinal-digit-suffix.md) | Ordinal Digit Suffix | **proposed** | none yet |
