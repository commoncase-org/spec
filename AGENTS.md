# AGENTS.md

## 1. System Metadata & Topology

- **Runtimes & Toolchain**: Python 3.12 (`.venv` via mise/uv), Bun
  1.4.0 (`knap@0.2.3`), `markdownlint-cli2` (v0.23.2).
- **Task Runner**: `mise` (see `mise.toml`).

```text
.
├── SPEC.md                    # Compiled normative standard (generated)
├── spec/
│   └── SPEC.template.md       # Source specification template (Knap)
├── data/
│   ├── transliteration.csv    # Normative 273-character mapping table
│   └── lexicon.json           # Normative Core Lexicon v1 table
├── conformance/
│   └── vectors.json           # Golden test vectors and decision links
├── decisions/                 # Architecture Decision Records (ADRs)
│   ├── 0000-template.md       # ADR template
│   └── 0001-*.md .. 0011-*    # Ratified decisions
├── scripts/
│   ├── build-spec             # Knap template compiler (Bun)
│   ├── validate               # Conformance & ADR validator (Python 3)
│   └── create-decision        # ADR scaffolding tool (Python 3)
├── .markdownlint-cli2.jsonc   # Lint config (strict 72-col line limit)
├── mise.toml                  # Tool pins and task runner definitions
└── package.json               # Bun dependencies (knap)
```

## 2. Deterministic Command Map

```sh
# Build specification from template via Knap
mise run build

# Verify SPEC.md is in sync with template (CI drift check)
bun scripts/build-spec --check

# Validate conformance/vectors.json and registries
mise run validate

# Lint all markdown files against 72-column line limit
mise run lint

# Scaffold a new ADR (auto-allocates next ID and slug)
mise run create-decision -- "<Decision Title>"
```

## 3. Architecture & Style Invariants

### Specification Authoring Standards (BCP-AGENTS-1 & RFC/ISO Norms)

- **Normative Language (BCP 14)**: Use `MUST`, `MUST NOT`, `SHOULD`,
  `SHOULD NOT`, `MAY` only in normative sections per RFC 2119 and
  RFC 8174. Capitalize strictly. Never use colloquial hedges.
- **Normative vs. Informative Separation**: Rules MUST be visually and
  structurally isolated from explanations and examples. Use single-line
  informative notes; never embed multi-line YAML example blocks into
  tokenizer rules.
- **Black-Box Conformance**: Conformance criteria (RFC 9110 §2.1) MUST
  evaluate observable input-to-output mapping, never internal white-box
  pipeline call stacks.
- **Formal Grammar**: Syntactic structures (tokens, delimiters,
  identifiers) MUST be defined in formal RFC 5234 ABNF.
- **Self-Contained Data**: Transliteration tables, lexicons, and test
  vectors MUST be declared in full within normative annexes, not
  external links.

### Template-Driven Build Architecture

- `SPEC.md` is a **compiled artifact**. All modifications to
  specification prose, grammar, or rules MUST be made in
  `spec/SPEC.template.md`.
- Tables in Annex A (Test Vectors), Annex B (Transliteration Table),
  and Annex C (Standard Core Lexicon) are mechanically imported and
  transformed from `conformance/vectors.json`,
  `data/transliteration.csv`, and `data/lexicon.json` using Knap
  filter pipelines:

  ```jinja2
  {{ "conformance/vectors.json" | include | parse_json | test_vectors_table }}
  {{ "data/transliteration.csv" | include | parse_csv | transliteration_table }}
  {{ "data/lexicon.json" | include | parse_json | lexicon_table }}
  ```

### Architectural Decision Records (ADRs)

- Every non-obvious normative rule, boundary decision, or error model
  MUST be grounded in an ADR in `decisions/`.
- Every ADR MUST follow `decisions/0000-template.md` and declare its
  associated test vector IDs in its `corpus: [...]` frontmatter.

### Markdown Formatting Invariants

- All Markdown prose, lists, and headings MUST wrap at column 72
  (enforced by `.markdownlint-cli2.jsonc` rule MD013).
- Tables and fenced code blocks are exempt from line wrapping.

## 4. Explicit Prohibitions (Negative Constraints)

- **NEVER** edit Annex A, Annex B, Annex C, or generated sections
  directly in `SPEC.md`. Edit `spec/SPEC.template.md`,
  `data/transliteration.csv`, `data/lexicon.json`, or
  `conformance/vectors.json`, then execute `mise run build`.
- **NEVER** use informal, speculative, or apologetic language in
  normative clauses (e.g., *"typically"*, *"in most cases"*,
  *"flatly wrong"*, *"asserted only by symmetry"*, *"proposed and not
  yet settled"*, *"the table currently defines"*).
- **NEVER** exceed 72 columns for prose lines in any markdown file.
- **NEVER** introduce natural-language symbol verbalization (e.g., `@`
  to `"at"`, `&` to `"and"`) into core commoncase; symbol translation
  belongs exclusively to upstream pre-processing (Unicode CLDR).
- **NEVER** use backtracking regular expressions with super-linear
  complexity ($O(2^n)$) for Rule T2 or delimiter tokenization; all
  algorithms MUST execute in $O(N)$ linear time.
- **NEVER** add runtime dependencies to `package.json` or `mise.toml`
  without explicit user confirmation.

## 5. Verification Pipeline (Definition of Done)

Prior to completing any task or proposing changes, execute the
following pipeline sequentially:

1. **Compile**: `mise run build`
   - Re-renders `SPEC.md` from `spec/SPEC.template.md` using Knap.
2. **Validate**: `mise run validate`
   - Validates all 5 serialization outputs across every vector in
     `conformance/vectors.json`.
   - Verifies that every decision slug referenced in
     `conformance/vectors.json` resolves to an existing file in
     `decisions/`.
   - Verifies schemas and readability of `data/lexicon.json` and
     `data/transliteration.csv`.
3. **Lint**: `mise run lint`
   - Validates all markdown files against MD013 (72-column limit) and
     markdownlint rules with zero issues.
4. **Git Cleanliness**: Check `git status` to ensure no stray scratch
   files or untracked temporary artifacts remain.

## 6. Anti-Patterns to Avoid

| Anti-Pattern | Operational Risk | Required Practice |
| :--- | :--- | :--- |
| **Direct Edit of `SPEC.md`** | Overwritten on next build; template drift. | Edit `spec/SPEC.template.md` and run `mise run build`. |
| **Inline Example Bloat** | 4:1 noise-to-signal ratio; obscures normative text. | Use concise single-line notes; group vectors in Annex A. |
| **Colloquial Normative Text** | Unenforceable standard; audit failures. | Use strict BCP 14 imperatives (`MUST`, `MUST NOT`). |
| **White-Box Conformance** | Bans optimized implementations (SIMD/DFAs). | Specify black-box observable input-to-output behavior. |
| **Unwrapped Markdown Lines** | Fails CI markdownlint checks. | Wrap all prose and headings strictly at column 72. |
