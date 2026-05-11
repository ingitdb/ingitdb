# Markdown Records with Frontmatter

## Problem Statement

How might we let inGitDB collections store records as `.md` files where YAML
frontmatter maps to declared columns and the document body maps to a content
column — so users can leverage Markdown-native tooling (Obsidian, Hugo, Jekyll,
code review on GitHub) while keeping inGitDB's schema and query semantics intact?

## Recommended Direction

Extend `RecordFileDef` with a new format value `markdown` and an optional
`content_field` key. Add an optional `format` property to `ColumnDef` so the
content column can be annotated as `format: markdown` (or anything else)
without inventing a new column type.

- **Frontmatter fields map flat**, no prefix. `field1: abc` in frontmatter
  becomes record field `field1`. No `frontmatter.` namespace — it adds zero
  semantic value and significant ergonomic cost in queries and code.
- **Schema validation stays strict**: only columns declared in the
  collection's `columns:` map are read from frontmatter. Unknown frontmatter
  keys are ignored (consistent with current YAML record behavior).
- **Document body maps to a content column**, named `$content` by default.
  Users can override via `record_file.content_field: <name>` when matching an
  SSG convention (Hugo `content`, Ghost `markdown`, etc.).
- The `$` sigil on the default name leverages inGitDB's existing reserved-name
  convention (`$records`, `$views`, `$record_id`) and prevents collisions with
  legitimate user-defined `content` / `body` frontmatter fields. `ColumnDef`
  does not currently validate column-name characters, so `$content` as a map
  key in `columns:` is admissible today.
- `RecordType` for markdown files is restricted to **`map[string]any`** in v1
  (one `.md` file = one record). List and map record types don't translate to
  Markdown semantically and are out of scope.
- **Byte-for-byte body preservation.** inGitDB is a low-level library and
  must not be smart about user content. The bytes between the closing `---`
  delimiter and end-of-file are the body verbatim — no trimming, no newline
  normalization, no smart-quote munging. This will be enshrined in the
  storage-format spec.
- **Column format hint, not column type.** Rather than introducing a
  `markdown` column type, the content column stays `type: string` and gains
  an optional `format: markdown` annotation via a new `ColumnDef.Format`
  field. This keeps the type system small and lets the same mechanism
  describe other string subspecies later (`format: html`, `format: json`,
  `format: uri`, …).
- **Default preview truncation: 60 characters, configurable.** When a content
  column shows up in `default_view` or README data preview, it's truncated to
  its first 60 characters by default. Authors override per-column via
  `ColumnDef.preview_max_length`, or per-view via a view-level setting.
- **File extension is recommended, not enforced.** `.md` is strongly
  recommended for external-tool compatibility (GitHub, IDEs, Obsidian) but
  the schema-declared `format: markdown` is authoritative. Non-`.md` names
  produce a warning, not an error.
- **Frontmatter key order is asymmetric.** The reader tolerates any
  frontmatter key order. The writer normalizes to `columns_order` whenever
  it writes; columns declared in the schema but absent from `columns_order`
  follow in alphabetical order. To avoid spurious reordering diffs when
  round-tripping a hand-authored file, the writer MUST NOT touch the file
  if no field values have changed (this is general to all inGitDB record
  formats and lives in the `storage-format` spec).

## Example

`blog_posts/.collection/definition.yaml`:

```yaml
record_file:
  name: "{key}.md"
  format: markdown
  type: map[string]any
  content_field: content    # optional; default is "$content"

columns:
  title:   { type: string, required: true }
  date:    { type: string }
  tags:    { type: string }
  content: { type: string, format: markdown }
```

`blog_posts/$records/hello-world.md`:

```markdown
---
title: Hello World
date: 2024-01-01
tags: intro,greeting
---
# Hello

This is the content of the post.
```

Read as record:

```yaml
title: Hello World
date: 2024-01-01
tags: intro,greeting
content: "# Hello\n\nThis is the content of the post.\n"
```

The trailing newline and any leading blank line after the closing `---` are
preserved verbatim. On rewrite, the writer reproduces the original body
bytes when the content field is unchanged.

## Key Assumptions to Validate

- [x] **YAML frontmatter (`---` delimiters) is sufficient for v1.** Confirmed.
      TOML and JSON frontmatter are out of scope.
- [x] **One `.md` file = one record is enough.** Confirmed.
      Record types other than `map[string]any` are not supported for markdown.
- [x] **`$content` as the default name is clear.** Sigil signals "synthetic /
      inGitDB-defined" consistently with existing project conventions.
- [ ] **An inline parser is sufficient.** Plan: implement a ~30-line parser
      first; reach for a library only when we hit a real edge case it solves
      (e.g. CRLF normalization, non-UTF-8 BOM handling) that we don't want to
      own.

## MVP Scope

**In:**

- New `RecordFormat` value: `markdown`.
- New optional field on `RecordFileDef`: `ContentField string` (default `$content`).
- New optional field on `ColumnDef`: `Format string`.
- Reader: parse `---`-delimited YAML frontmatter, validate keys against declared columns, map body bytes to the content column verbatim.
- Writer: serialize declared frontmatter columns to YAML between `---` delimiters, write content column bytes as body without modification.
- Validation:
  - error if `format: markdown` is paired with a `RecordType` other than `map[string]any`;
  - error if `content_field` is set but the named column isn't declared;
  - error if `content_field` is set on a non-markdown format.
- Preview truncation default: 60 characters.
- Tests: round-trip parse/serialize (byte-identical); edge cases (empty body, empty frontmatter, multiple `---` in body, CRLF input).
- Fixture collection added to `demo-dbs/`.

**Out:**

- DALgo CRUD changes beyond what `map[string]any` already provides (it reuses the same code path).
- TOML / JSON / custom frontmatter delimiters.
- Schema-driven frontmatter-key → column-name remapping.
- TUI changes beyond rendering a truncated preview.
- Markdown rendering. inGitDB stores the bytes; rendering is the caller's job.

## Not Doing (and Why)

- **`frontmatter.` field prefix** — adds ergonomic cost everywhere with no semantic value. The `$` sigil on the content column already provides the only namespace separation actually needed.
- **TOML frontmatter (`+++`)** — minority format. YAML covers >95% of real-world `.md` files. Can be added later behind a `delimiter` option if demand emerges.
- **`[]map[string]any` or `map[string]map[string]any` record types in Markdown** — no natural Markdown representation. Forcing one would be unprincipled.
- **Schema-driven frontmatter remapping** — too much config surface for v1.
- **Auto-deriving a content column** if the user omits it — silent magic. Better to error: "`content_field: X` references undeclared column."
- **Body normalization** (trim trailing whitespace, ensure single final newline) — would defeat byte-for-byte preservation and create Git diffs the user didn't author.
- **A `markdown` column type** — premature taxonomy. `string` + `format: markdown` is a smaller wedge that generalizes to other subspecies later.

## Open Questions

- ~~Frontmatter library choice~~ — **resolved.** Start with an inline parser; revisit only if a concrete need (CRLF normalization, BOM handling, etc.) emerges that we don't want to own.
- ~~Body normalization policy~~ — **resolved.** Byte-for-byte preservation. Documented in the storage-format spec.
- ~~Markdown column type vs format property~~ — **resolved.** Use `ColumnDef.Format` (`format: markdown`).
- ~~Long-content preview behavior~~ — **resolved.** Default truncation: 60 characters.
- ~~`ColumnDef.Format` free-form vs controlled vocabulary~~ — **resolved.** Well-known list (`markdown`, `html`, `json`, `jsonl`, `yaml`, `uri`, `email`, `pdf`) with free-form fallback. TS-shaped: `type Format = "markdown" | "html" | ... | (string & {})`.
- ~~Required-field-missing policy~~ — **resolved** and promoted to `collection-schema` (cross-cutting, applies to all record formats). Write fails with an error naming the column; read succeeds but surfaces the validation error per-record (DALgo `record.SetError(err)`).
- ~~Preview truncation default value~~ — **resolved.** 60-character default, configurable per-column via `ColumnDef.preview_max_length`, with optional view-level override.
- ~~`no-rewrite-without-change` promotion~~ — **resolved.** Promoted into `storage-format` as `REQ: no-rewrite-without-change` with `AC: no-op-round-trip-clean`; applies to all record formats.
- ~~Tie-breaker order for unordered columns~~ — **resolved.** Alphabetical by column name.
