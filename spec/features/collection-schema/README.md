# Feature: Collection Schema

**Status:** Draft

## Summary

Every collection has a declarative schema in `.collection/definition.yaml` that defines its columns, their types, the record file storage layout, and optional metadata such as localized titles and view configuration. The schema is the authoritative description of what records the collection may hold.

## Problem

Without a declared schema, a directory of YAML or JSON files is just a heap of documents. There is nothing for tooling to validate against, no way to know which fields are required, and no way for downstream consumers to rely on a stable shape. A schema lets validation, materialization, and code generation share a single source of truth.

## Behavior

### Schema location

Each collection's schema lives in `.collection/definition.yaml` inside the collection directory. The corresponding JSON Schema for this file is published in the `ingitdb-schema` repository as `ingitdb-collection.schema.json`.

#### REQ: definition-file-location

A collection's schema MUST live at `<collection-dir>/.collection/definition.yaml`. No other location is recognized.

#### REQ: schema-conforms-to-json-schema

The contents of `definition.yaml` MUST conform to the published `ingitdb-collection.schema.json` JSON Schema. Tools MAY validate `definition.yaml` against this schema as a precondition for further processing.

### Columns

A schema declares a map of column name to column definition. Each column has a type and optional metadata such as `foreign_key`.

#### REQ: columns-required

A collection schema MUST declare at least one column. The column set is the contract that records are validated against.

#### REQ: column-type

Each column MUST declare a `type`. Recognized types include scalars (e.g. `string`, `int`, `bool`) and structured types as defined by `ingitdb-schema`.

### Record file declaration

The schema declares how records are laid out on disk via a `record_file.type` field.

#### REQ: record-file-type-declared

Every collection schema MUST declare a `record_file.type` selecting one of the supported record file types (see [record-file-types](../record-file-types/README.md)).

### Optional metadata

Schemas may carry additional metadata: localized titles, default views, ordering, FK columns. These are optional and do not affect record validity.

## Dependencies

- collection
- record-file-types

## Acceptance Criteria

### AC: well-formed-schema

**Requirements:** collection-schema#req:definition-file-location, collection-schema#req:schema-conforms-to-json-schema, collection-schema#req:columns-required, collection-schema#req:record-file-type-declared

A `.collection/definition.yaml` that exists at the right path, validates against `ingitdb-collection.schema.json`, declares at least one column, and selects a valid `record_file.type` is accepted.

### AC: schema-mismatch-rejected

**Requirements:** collection-schema#req:column-type, collection-schema#req:record-file-type-declared

A schema missing required fields (no columns, missing `record_file.type`, or columns without `type`) is rejected by validation, with the offending field reported.

## Outstanding Questions

- Which scalar and structured column types are part of the stable contract versus extensible by future schema versions?
- How are schema versions handled when the published JSON Schema evolves — does the file declare its target schema version?

---
*This document follows the https://specscore.md/feature-specification*
