# commoncase Specification

## Preface (informative)

This specification defines **commoncase**, an unambiguous,
deterministic standard for the transformation of singular identifier
strings into canonical case representations.

Processing is governed by an invariant, four-stage sequential
pipeline: Normalization, Tokenization, Case Folding, and
Serialization. Execution order is strictly monotonic; no stage may
be skipped, reordered, or executed conditionally.

Every normative rule below is illustrated by at least one worked
example. This document is self-contained: its Conformance clause and
test vectors are fully defined within it, and no external file is
required to implement or verify it.

## References

### Normative References

- **[BCP14]** Bradner, S., "Key words for use in RFCs to Indicate
  Requirement Levels", BCP 14, RFC 2119, March 1997,
  <https://www.rfc-editor.org/info/bcp14>.

- **[ICAO9303]** International Civil Aviation Organization, "Machine
  Readable Travel Documents", Doc 9303, Part 3, Section 6,
  <https://www.icao.int/publications/doc-series/doc-9303>.

## Terminology

The following terms are used throughout this specification with the
meanings given here. No other term is used as a synonym for any of
these.

**Token** - the smallest unit of meaning that Tokenization identifies
within an input string. Case Folding and Serialization operate only
on Tokens; neither stage ever inspects the original input string.

**Boundary** - a position between two characters of an input string
at which Tokenization divides the string into two separate Tokens.

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
Tokenization's mechanical output for a specific string, applied
before Case Folding runs.

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
characters beyond the mapping table currently defined in
Normalization; or a settled policy for treating trademarked compounds
as atomic.

## Conformance

An implementation of this specification conforms if and only if it:

1. performs Normalization, then Tokenization, then Case Folding, then
   Serialization, in that order, with no stage skipped or reordered;
   and
2. produces output matching every test vector in Annex A for at
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

Normalization **MUST** transliterate non-ASCII characters to ASCII
before Tokenization runs, using a closed, deterministic mapping table
sourced from **[ICAO9303]**. The table currently defines the entries
**[ICAO9303]** assigns to German: a with an umlaut to "ae", o with an
umlaut to "oe", u with an umlaut to "ue", and the eszett character to
"ss". After Normalization, the string **MUST** contain only ASCII
characters.

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

Tokenization **MUST** divide a Normalized input string into a
sequence of Tokens. It does this in two steps: first, the
Orthographic Segmentation Rules mechanically identify Boundaries;
second, the Lexicon **MAY** override the mechanical result for
specific strings before Case Folding runs.

### Orthographic Segmentation Rules

The following rules, labeled T1 through T5 with one lettered
sub-rule, govern where Tokenization places a Boundary based on
adjacent characters.

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

**T2** - A Cap-run followed by a lowercase letter **MUST** have a
Boundary placed immediately before the Cap-run's final letter, so
that the final letter joins the following lowercase run instead of
the Cap-run.

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

**T3** - A transition from a letter to a digit **MUST NOT** be
treated as a Boundary; the digit fuses onto the preceding letters.

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
treated as a Boundary.

```example
  input: Iso8601Date
  tokens: iso8601, date
  pascal: Iso8601Date
  camel: iso8601Date
  snake: iso8601_date
  kebab: iso8601-date
  constant: ISO8601_DATE
```

**T5** - A transition from a digit to a lowercase letter **MUST NOT**
be treated as a Boundary. This sub-case is unvalidated: no test
vector in Annex A exercises a digit immediately followed by a
lowercase letter, and it is asserted only by symmetry with T3.

### Default Segmentation Behavior

A Delimiter-free run **MUST** default to a single Token when it
contains no Orthographic signal for the Orthographic Segmentation
Rules to act on. This default **MUST NOT** be overridden by
heuristic dictionary segmentation; it **MAY** be overridden only by
an explicit Lexicon entry.

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

- to correct segmentation that the Orthographic Segmentation Rules
  get flatly wrong, such as `oauth2` or `graphql`, which do not
  decompose into their intended sub-words mechanically;
- to force or confirm a split within a Delimiter-free run, where the
  default of one Token is not the intended reading; and
- to force a mechanically correct split to remain one Token for
  trademark reasons. This use is proposed and not yet a settled part
  of this specification.

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

Case Folding **MUST** capitalize the first letter of a Token and
lowercase its remaining letters, for every style except constant.
This rule applies uniformly to every Token regardless of length or
origin - no exception is made for a Token that happens to be an
acronym, and none is made for a Token supplied by the Lexicon rather
than by the Orthographic Segmentation Rules.

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

Serialization **MUST** produce each of the following five styles
from a Token sequence, using the case forms Case Folding has already
applied:

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
Pascal, except the first Token's leading letter **MUST** remain
lowercase.

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

## Annex A (informative) Test Vectors

This annex is informative. It is the complete, self-contained set of
test vectors referenced by the Conformance clause; an implementation
need not consult any file outside this specification to be tested
against it.

| ID | Input | Tokens | Pascal | camel | snake | kebab | constant |
|---|---|---|---|---|---|---|---|
| 001 | `userId` | user, id | `UserId` | `userId` | `user_id` | `user-id` | `USER_ID` |
| 002 | `IPAddress` | ip, address | `IpAddress` | `ipAddress` | `ip_address` | `ip-address` | `IP_ADDRESS` |
| 003 | `DBConnectionPool` | db, connection, pool | `DbConnectionPool` | `dbConnectionPool` | `db_connection_pool` | `db-connection-pool` | `DB_CONNECTION_POOL` |
| 004 | `IOStream` | io, stream | `IoStream` | `ioStream` | `io_stream` | `io-stream` | `IO_STREAM` |
| 005 | `HTTPRequestHandler` | http, request, handler | `HttpRequestHandler` | `httpRequestHandler` | `http_request_handler` | `http-request-handler` | `HTTP_REQUEST_HANDLER` |
| 006 | `XMLHttpRequest` | xml, http, request | `XmlHttpRequest` | `xmlHttpRequest` | `xml_http_request` | `xml-http-request` | `XML_HTTP_REQUEST` |
| 007 | `Sha256Hash` | sha256, hash | `Sha256Hash` | `sha256Hash` | `sha256_hash` | `sha256-hash` | `SHA256_HASH` |
| 008 | `Aes256Key` | aes256, key | `Aes256Key` | `aes256Key` | `aes256_key` | `aes256-key` | `AES256_KEY` |
| 009 | `Utf8String` | utf8, string | `Utf8String` | `utf8String` | `utf8_string` | `utf8-string` | `UTF8_STRING` |
| 010 | `Iso8601Date` | iso8601, date | `Iso8601Date` | `iso8601Date` | `iso8601_date` | `iso8601-date` | `ISO8601_DATE` |
| 011 | `top10Items` | top10, items | `Top10Items` | `top10Items` | `top10_items` | `top10-items` | `TOP10_ITEMS` |
| 012 | `IPv6Address` | ipv6, address | `Ipv6Address` | `ipv6Address` | `ipv6_address` | `ipv6-address` | `IPV6_ADDRESS` |
| 013 | `OAuth2Client` | oauth2, client | `Oauth2Client` | `oauth2Client` | `oauth2_client` | `oauth2-client` | `OAUTH2_CLIENT` |
| 014 | `GraphQLSchema` | graphql, schema | `GraphqlSchema` | `graphqlSchema` | `graphql_schema` | `graphql-schema` | `GRAPHQL_SCHEMA` |
| 015 | `iPhone` | i, phone | `IPhone` | `iPhone` | `i_phone` | `i-phone` | `I_PHONE` |
| 016 | `checkIOSVersion` | check, ios, version | `CheckIosVersion` | `checkIosVersion` | `check_ios_version` | `check-ios-version` | `CHECK_IOS_VERSION` |
| 017 | `filepath` | file, path | `FilePath` | `filePath` | `file_path` | `file-path` | `FILE_PATH` |
| 018 | `username` | username | `Username` | `username` | `username` | `username` | `USERNAME` |
| 019 | `metadata` | metadata | `Metadata` | `metadata` | `metadata` | `metadata` | `METADATA` |
| 020 | `xCoordinate` | x, coordinate | `XCoordinate` | `xCoordinate` | `x_coordinate` | `x-coordinate` | `X_COORDINATE` |
| 021 | `straße` | strasse | `Strasse` | `strasse` | `strasse` | `strasse` | `STRASSE` |
| 022 | `müllerStraße` | mueller, strasse | `MuellerStrasse` | `muellerStrasse` | `mueller_strasse` | `mueller-strasse` | `MUELLER_STRASSE` |
