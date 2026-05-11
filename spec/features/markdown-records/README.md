# Feature: Markdown Records

**Status:** Draft

## Summary

A collection MAY declare its record file format as `markdown`, in which case
records are stored as `.md` files with YAML frontmatter delimited by `---`.
Frontmatter keys map flat to columns declared in the collection schema; the
document body — every byte between the closing `---` and end-of-file — maps to
a single configurable content column, named `$content` by default. The body
is preserved byte-for-byte by both reader and writer.

## Problem

Markdown with YAML frontmatter is the de-facto storage format for
human-authored documents in static site generators (Hugo, Jekyll, Astro),
knowledge bases (Obsidian), and content pipelines (Ghost, Gatsby, gray-matter).
Forcing such content into pure YAML files would either drop the body content
entirely or quote it as a string field with backslash-escaped newlines —
unreadable in any text editor, useless in a `git diff`, hostile to authors.

Supporting `.md` files as a first-class record format lets inGitDB consume and
produce content that downstream Markdown tooling already understands, while
keeping the schema, validation, and query semantics of a typed collection.

## Behavior

### Format declaration

A collection opts into Markdown storage by setting `record_file.format` to
`markdown` in `.collection/definition.yaml`.

#### REQ: format-value

When a collection declares `record_file.format: markdown`, its record files
MUST use the layout defined in this feature: a `.md` file containing optional
YAML frontmatter delimited by `---` lines, followed by a document body.

#### REQ: file-extension-recommended

Record files for a markdown-format collection SHOULD use the `.md` extension
so that external tools (GitHub, IDEs, Obsidian, static site generators) render
them as Markdown. The format is determined by the schema, not by the file
extension, so inGitDB MUST NOT reject a `record_file.name` template that
yields a different (or absent) extension; it MAY emit a warning.

#### REQ: record-type-restriction

A markdown-format collection MUST declare `record_file.type: map[string]any`.
The list (`[]map[string]any`) and dictionary (`map[string]map[string]any`)
record file types are not supported for `markdown` format and MUST be
rejected by validation.

### Frontmatter

Frontmatter is the optional block at the start of the file delimited by
matching `---` lines. Its content is parsed as YAML and contributes
key/value pairs to the record.

#### REQ: frontmatter-delimiters

When present, frontmatter MUST be introduced by a line containing exactly
`---` as the very first line of the file (no preceding blank lines, no BOM
between the start of file and the delimiter) and closed by a subsequent line
containing exactly `---`. A file with no opening `---` on its first line has
no frontmatter; its entire contents are the body.

#### REQ: frontmatter-yaml

The content between the opening and closing `---` lines MUST be valid YAML
and MUST parse to a YAML mapping (`map[string]any`). A YAML scalar, sequence,
or non-mapping document at the top of frontmatter is a validation error.

#### REQ: frontmatter-flat-mapping

Each top-level key in the frontmatter mapping is treated as a record field
named exactly as it appears — no prefix, no namespace. inGitDB MUST NOT inject
synthetic namespaces (e.g. `frontmatter.`) into field names.

#### REQ: frontmatter-strict-columns

Only frontmatter keys that match a column declared in the collection schema
contribute to the record. Frontmatter keys that do not match any declared
column are ignored (consistent with existing YAML record loading).

### Body and content column

The body is the byte range from immediately after the line terminator of the
closing `---` to end-of-file. It is exposed as a single record field whose
name is determined by `record_file.content_field`.

#### REQ: content-field-default

When `record_file.format` is `markdown` and `record_file.content_field` is
not set, the body MUST be exposed under the field name `$content`. The
collection schema MUST declare a column with that name.

#### REQ: content-field-override

When `record_file.content_field` is set, the body MUST be exposed under the
named field. The collection schema MUST declare a column with that exact
name; otherwise the schema is rejected.

#### REQ: content-field-restricted-to-markdown

`record_file.content_field` MUST only appear when `record_file.format` is
`markdown`. Setting it for any other format is a schema validation error.

#### REQ: body-byte-fidelity

The reader MUST expose the body as the verbatim byte sequence between the
closing `---` line terminator and end-of-file, with no trimming, no newline
normalization (LF/CRLF preserved as-is), and no character substitution.

#### REQ: body-write-fidelity

When writing a record back to disk, the writer MUST emit the content field's
bytes verbatim as the body, after the closing `---` line. Writing a record
whose body field has not been modified MUST produce a byte-identical body
section (round-trip stability).

### Frontmatter key order

Frontmatter key ordering is asymmetric: the reader tolerates any order, but
the writer normalizes to a canonical order so written files are deterministic
and Git diffs are predictable.

#### REQ: frontmatter-read-tolerates-any-order

The reader MUST accept frontmatter whose keys appear in any order. The order
of keys in the source file MUST NOT cause a parse error or affect the
resulting record values.

#### REQ: frontmatter-write-uses-columns-order

When the writer emits frontmatter, it MUST order the keys to match the
collection's `columns_order` for the columns that appear in it. Columns
declared in the schema but absent from `columns_order` MUST follow the
ordered keys in alphabetical order by column name. This rule yields a
single canonical ordering shared by every inGitDB implementation.

#### REQ: write-reorders-on-value-change

When at least one field value is changed, the writer MAY (and typically
will) emit frontmatter in canonical `columns_order`, which can reorder keys
relative to the source file. This is accepted as a one-time normalization
on edit and is not a violation of byte-fidelity (which applies only to the
body, see `body-write-fidelity`).

### Column metadata

The content column is a regular `string` column. To let downstream consumers
distinguish a Markdown body from an arbitrary string, columns gain an
optional `format` annotation.

#### REQ: column-format-property

A `ColumnDef` MAY declare an optional `format` property carrying a string
identifier that hints at the column's logical content type. inGitDB
recognizes a set of well-known values that tooling SHOULD interpret
consistently, and accepts any other string as a free-form extension:

- Well-known values: `markdown`, `html`, `json`, `jsonl`, `yaml`, `uri`,
  `email`, `pdf`. Tooling SHOULD render/preview these consistently across
  implementations.
- Any other string is accepted and passed through as-is. inGitDB MUST NOT
  reject an unknown `format` value.

Conceptually equivalent to a TypeScript signature like
`type Format = "markdown" | "html" | "json" | "jsonl" | "yaml" | "uri" | "email" | "pdf" | (string & {})`.

The `format` property is metadata: inGitDB does not validate the column's
stored value against it. Tooling MAY use it to choose a renderer, a preview
strategy, or a validator.

#### REQ: content-column-format-hint

When a column is referenced by `record_file.content_field` (or named
`$content` under the default), authors SHOULD annotate it with
`format: markdown`. inGitDB does not require the annotation.

### Preview truncation

Long content columns are unwieldy in tabular previews. A default truncation
length keeps previews readable without losing the field.

#### REQ: content-preview-truncation-default

When the content column is rendered inside a `default_view` or a README data
preview, it MUST be truncated, with an indicator that truncation occurred
(e.g. `…`). The truncation length is resolved in this precedence order:

1. View-level override (`ViewDef`-scoped per-column setting), if present.
2. Column-level setting on `ColumnDef` (`preview_max_length: <N>`), if set.
3. Default: 60 characters.

The default value (60) is a project-wide constant and not a per-collection
setting. Implementations MAY expose it through a configuration knob for
future tuning but MUST start at 60.

### Validation summary

A markdown-format schema is rejected if any of the following hold:

- `record_file.type` is not `map[string]any`;
- `record_file.content_field` names a column not declared in `columns`;
- `record_file.content_field` is set when `record_file.format` is not `markdown`;
- frontmatter in a record file is not a YAML mapping.

A markdown-format schema MAY emit a warning (but not an error) if the
`record_file.name` template does not yield a `.md` filename.

## Dependencies

- collection-schema
- record-file-types
- storage-format

## Acceptance Criteria

### AC: markdown-collection-roundtrip

**Requirements:**
markdown-records#req:format-value,
markdown-records#req:file-extension-recommended,
markdown-records#req:record-type-restriction,
markdown-records#req:frontmatter-delimiters,
markdown-records#req:frontmatter-yaml,
markdown-records#req:frontmatter-flat-mapping,
markdown-records#req:frontmatter-strict-columns,
markdown-records#req:content-field-default,
markdown-records#req:content-field-override,
markdown-records#req:body-byte-fidelity,
markdown-records#req:body-write-fidelity

A markdown-format collection containing a `.md` record with valid YAML
frontmatter and a non-empty body can be read into a typed record whose
declared columns are populated from the frontmatter and whose content column
holds the body bytes verbatim; rewriting the same record without modifying
the content field produces a byte-identical body section on disk.

### AC: frontmatter-key-ordering

**Requirements:**
markdown-records#req:frontmatter-read-tolerates-any-order,
markdown-records#req:frontmatter-write-uses-columns-order,
markdown-records#req:write-reorders-on-value-change

A `.md` file whose frontmatter keys appear in arbitrary order (not matching
`columns_order`) is read without error. After modifying any field value and
writing the record back, the resulting file's frontmatter keys appear in the
order declared by `columns_order` (with any unordered columns following in a
deterministic order).

### AC: no-rewrite-when-unchanged

**Requirements:** storage-format#req:no-rewrite-without-change

Reading a `.md` record and writing it back with no modifications to any
field value produces no change to the file on disk: a `git status` after
the round-trip shows the working tree clean, even if the source file's
frontmatter was in non-canonical order. (Inherits the general
`no-rewrite-without-change` rule from `storage-format`.)

### AC: markdown-schema-validation

**Requirements:**
markdown-records#req:record-type-restriction,
markdown-records#req:content-field-override,
markdown-records#req:content-field-restricted-to-markdown

A schema that pairs `format: markdown` with a non-`map[string]any` record
type, or sets `content_field` to a column that isn't declared, or sets
`content_field` on a non-markdown format, is rejected by validation with
the offending field reported.

### AC: column-format-metadata

**Requirements:**
markdown-records#req:column-format-property,
markdown-records#req:content-column-format-hint

A `ColumnDef` carrying `format: markdown` is accepted by validation, exposed
to tooling, and does not affect record loading or storage semantics.

### AC: content-preview-truncation

**Requirements:** markdown-records#req:content-preview-truncation-default

A markdown record whose body exceeds 60 characters renders in a default
view or README preview as the first 60 characters of the body followed by
a truncation indicator, unless the view declares an explicit override.

## Outstanding Questions

- None at the spec level. Required-field handling has been promoted to
  `collection-schema` (asymmetric: write fails, read succeeds with a
  per-record error). Preview truncation is now configurable per-column with
  a 60-character default. `ColumnDef.format` is a well-known list with
  free-form fallback.

Implementation-time decisions deferred to first code review:

- Exact YAML key name for the per-column preview length setting
  (`preview_max_length` proposed in this doc, may be revised).
- Exact YAML key name for the view-level override.

---
*This document follows the https://specscore.md/feature-specification*
