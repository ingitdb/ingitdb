# Feature: Record File Types

**Status:** Draft

## Summary

A collection's `record_file.type` selects one of three on-disk layouts for its records: one file per record (`map[string]any`), a single list file containing all records (`[]map[string]any`), or a single dictionary file keyed by record ID (`map[string]map[string]any`). Each type implies a distinct directory layout and Git diff behavior.

## Problem

Different shapes of data benefit from different physical layouts. A large, slowly-changing reference table (countries) is better as one file per record so each record has its own commit history. A small fast-changing list (build statuses) is better as a single list file. A dictionary keyed by ID is convenient for lookup-style data. Forcing all collections into one layout would either explode the file count or wreck per-record history. The schema picks the right layout per collection.

## Behavior

### `map[string]any` — one file per record

Each record is stored in its own file inside the collection directory. The file's name is the record's ID; its contents are the record's fields as a map.

```
companies/
  acme.yaml       # {name: Acme, country: gb}
  shopify.yaml    # {name: Shopify, country: ca}
```

#### REQ: map-string-any-layout

When `record_file.type` is `map[string]any`, each record MUST be stored in a separate file under the collection directory. The file name (without extension) is the record ID.

#### REQ: map-string-any-crud

`map[string]any` collections support full record-level CRUD operations because each record maps to a single file with its own commit history.

### `[]map[string]any` — single list file

All records are stored in a single list (array) inside one file. The list order is meaningful for record identity within the file.

#### REQ: list-layout

When `record_file.type` is `[]map[string]any`, the collection MUST store its records as a single list inside one file. There is no per-record file.

### `map[string]map[string]any` — single dictionary file

All records are stored as a dictionary in a single file, keyed by record ID. Lookup by ID is direct; the file is rewritten as a whole on any change.

#### REQ: dictionary-layout

When `record_file.type` is `map[string]map[string]any`, the collection MUST store its records as a single map keyed by record ID inside one file.

### Diff and history implications

The choice of record file type has direct consequences for Git history granularity.

#### REQ: history-granularity

Per-record commit history is only available with the `map[string]any` type. The list and dictionary types have collection-level history only — every change touches the single file containing all records.

### Excluding non-record files

A directory that holds `map[string]any` records may legitimately contain auxiliary files alongside records — a `README.md` documenting the collection, a `.gitkeep`, an `index.md` rendered by a static site generator. With the file extension alone as the record signal, these files would be misclassified as records and break reads, counts, and validation.

`record_file.exclude_regex` is an optional regular expression applied to the basename of each candidate record file. Files whose basename matches MUST be omitted from reads, validation, and record counts.

#### REQ: exclude-regex-optional

A collection schema MAY declare `record_file.exclude_regex` as a regular expression string. The pattern MUST compile under Go's `regexp` package (RE2 syntax); a schema with an uncompilable pattern MUST be rejected by validation.

#### REQ: exclude-regex-basename-scope

When `exclude_regex` is set, implementations MUST apply the pattern to each candidate file's basename (the last path segment), not to its full path. The match is partial by default (Go `regexp.MatchString` semantics); authors anchor with `^` and `$` when they want full-name matching.

#### REQ: exclude-regex-uniform-application

A file whose basename matches `exclude_regex` MUST be excluded uniformly from:

1. record reads (the file is invisible to per-key `Get` and to query enumeration);
2. record validation (validators MUST NOT report errors against excluded files);
3. record counts (excluded files MUST NOT contribute to per-collection record counts).

Inconsistent exclusion across these paths is a defect.

#### REQ: exclude-regex-empty-default

When `exclude_regex` is empty or omitted, no files are excluded by this rule. Hidden files (names starting with `.`) and reserved names (`$records`, `$ingitdb`, etc.) remain excluded by their own existing rules.

## Dependencies

- collection-schema
- storage-format

## Acceptance Criteria

### AC: type-determines-layout

**Requirements:** record-file-types#req:map-string-any-layout, record-file-types#req:list-layout, record-file-types#req:dictionary-layout

A collection's directory layout matches its declared `record_file.type`. A collection that declares `map[string]any` but stores records in a single file (or vice versa) fails validation.

### AC: crud-restricted-to-map-string-any

**Requirements:** record-file-types#req:map-string-any-crud, record-file-types#req:history-granularity

Per-record CRUD operations succeed for `map[string]any` collections. Attempting per-record CRUD against `[]map[string]any` or `map[string]map[string]any` is either rejected or implemented at collection-file granularity.

### AC: exclude-regex-omits-auxiliary-files

**Requirements:** record-file-types#req:exclude-regex-optional, record-file-types#req:exclude-regex-basename-scope, record-file-types#req:exclude-regex-uniform-application

A `map[string]any` collection that places a `README.md` in its records directory and declares `record_file.exclude_regex: '^README\.md$'` does not return that file as a record from queries, does not count it toward `total_records`, and does not flag it as a validation error — while still returning and counting all `{key}.md` records that don't match.

## Outstanding Questions

- For `map[string]any`, what is the canonical extension policy when both YAML and JSON files appear in the same collection directory?
- Should `map[string]map[string]any` allow ordering metadata, or is the dictionary type explicitly unordered?

---
*This document follows the https://specscore.md/feature-specification*
