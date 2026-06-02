# Feature: Storage Format

> [SpecScore.**Studio**](https://specscore.studio): | [Explore](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/storage-format?op=explore) | [Edit](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/storage-format?op=edit) | [Ask question](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/storage-format?op=ask) | [Request change](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/storage-format?op=request-change) |
**Status:** Draft

## Summary

inGitDB stores every record as a plain text file on disk encoded as YAML, JSON, TOML, Markdown-with-frontmatter, or INGR (in-Git Records). The format is human-readable, Git-diffable, and contains no proprietary binary structures, indexes, or daemon-managed state. A clone of the repository is a complete copy of the database.

## Problem

Traditional databases store data in opaque binary files that cannot be reviewed in a pull request, edited in a text editor, or diffed line by line. Teams that already manage code through Git want the same workflows — branches, reviews, blame — for their data. That requires a storage format that is itself a first-class citizen of a Git repository, not a binary blob committed to one.

## Behavior

### File encoding

Records are stored as text files using one of the supported text encodings. The choice is per-collection and recorded in the collection schema. No proprietary binary encoding, no compressed pack file, and no sidecar index is required to read a record.

#### REQ: text-only

Record files MUST be encoded as UTF-8 text. Binary record content is not supported.

#### REQ: supported-formats

A collection MUST declare its record format as one of: `yaml` (or its alias `yml`), `json`, `toml`, `markdown`, or `ingr`. Mixing formats within a single collection is not permitted. Binary serializations (CBOR, MessagePack, BSON, JSONB, Protobuf, Avro, etc.) are out of scope because they violate `text-only` and `editor-readable`.

#### REQ: editor-readable

Record files MUST be readable and editable in any standard text editor without specialized tooling. Layout choices that defeat human review (single-line minified JSON for large records, embedded base64 blobs as a default) are out of scope for this format.

### Git-diffability

The format is optimized for Git's line-based diff. Records are laid out one logical field per line where the encoding allows, so that a change to a single field produces a single-line diff in a pull request.

#### REQ: line-oriented-diffs

Record files SHOULD be formatted such that a small logical change (e.g. updating one field) produces a small textual diff. The default writers MUST emit records in a stable, deterministic key order across all supported formats.

#### REQ: deterministic-serialization

When inGitDB tooling writes a record file, the serialization MUST be deterministic across runs given the same input. Reordering keys between writes is a defect.

#### REQ: no-rewrite-without-change

When a write operation's resulting record values are identical to what was last read from disk, the writer MUST NOT modify the file. This prevents spurious whitespace, key-order, or formatting diffs from round-tripping a hand-authored file through inGitDB tooling. The rule applies to every record file format (YAML, JSON, TOML, Markdown, INGR, future additions): a no-op round-trip MUST leave the working tree clean under `git status`.

### No proprietary index

Reading a record does not require a database server, daemon, or rebuilt index. The on-disk file is the source of truth.

#### REQ: no-daemon-required

A consumer with a local clone MUST be able to read any record by reading the file directly. Auxiliary artifacts (materialized views, caches) are derived and MUST NOT be required for correctness of a read.

## Dependencies

- collection
- record-file-types

## Acceptance Criteria

### AC: format-is-text

**Requirements:** storage-format#req:text-only, storage-format#req:supported-formats, storage-format#req:editor-readable

A repository whose record files are valid UTF-8 in one of the supported text formats (YAML, JSON, TOML, Markdown, INGR), readable in any text editor, satisfies the storage format. A repository containing a binary record file or a record encoded in a format outside the supported set is rejected.

### AC: deterministic-writes

**Requirements:** storage-format#req:line-oriented-diffs, storage-format#req:deterministic-serialization

Writing the same logical record twice produces byte-identical files. A small field change produces a small diff under standard `git diff` settings.

### AC: no-op-round-trip-clean

**Requirements:** storage-format#req:no-rewrite-without-change

Reading any record file and writing it back without modifying any field value produces no change on disk. A `git status` after the round-trip shows the working tree clean, regardless of whether the source file used canonical key ordering or formatting.

## Open Questions

- ~~Should a third encoding (e.g. TOML) be reserved for future use, or is the YAML/JSON choice intentionally final?~~ — **resolved.** TOML is supported alongside YAML and JSON. Markdown is also supported as a structured format with YAML frontmatter; see `markdown-records`.
- What is the policy for embedded large blobs — link out to a separate file or accept inline base64?

---
*This document follows the https://specscore.md/feature-specification*
