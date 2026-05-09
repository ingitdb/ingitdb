# Feature: Collection

**Status:** Draft

## Summary

A collection is a directory of records validated against a single schema. Collections are addressed by an ID composed of alphanumerics and dots, and they may nest: a record in one collection can itself contain sub-collections, allowing naturally hierarchical data without joins or foreign keys.

## Problem

Records on disk need a unit of organization that is large enough to share a schema and small enough to be addressable. Flat tables work for shallow data, but real-world reference data is often hierarchical — countries contain regions, regions contain cities. Without first-class hierarchy, the format would force every relationship through foreign keys, defeating the readability that comes from nested directories.

## Behavior

### Collection identity

Every collection has a globally unique ID within a database. The ID is the addressable handle used by configuration, CLI, and tooling.

#### REQ: id-charset

A collection ID MUST consist of alphanumeric characters and the `.` separator only. It MUST start and end with an alphanumeric character. Underscores, hyphens, slashes, and other punctuation are not permitted.

#### REQ: id-uniqueness

A collection ID MUST be unique within its containing database, including IDs introduced via namespace imports.

### Collection directory layout

A collection lives in a directory containing both schema metadata and record files.

```
companies/
  .collection/
    definition.yaml
  acme.yaml
  shopify.yaml
```

#### REQ: collection-directory

Every collection MUST be a directory on disk containing a `.collection/` sub-directory with a `definition.yaml` file. Records live alongside `.collection/` according to the collection's `record_file.type`.

#### REQ: schema-binding

All record files inside a collection directory MUST conform to that collection's schema. A collection has exactly one schema; records that do not conform are validation errors.

### Sub-collections

A record may contain its own collections. Sub-collections are full collections with their own `.collection/definition.yaml` and follow the same rules as root collections.

#### REQ: sub-collections-allowed

A collection MAY contain sub-collections nested within individual record directories. A sub-collection MUST itself satisfy every requirement of this feature, including having a `.collection/definition.yaml`.

#### REQ: hierarchical-addressing

Nested collections are addressed by extending the parent record's path with the sub-collection ID. The dot in a collection ID is a logical separator, not a directory separator.

## Dependencies

- collection-schema
- storage-format

## Acceptance Criteria

### AC: id-validation

**Requirements:** collection#req:id-charset, collection#req:id-uniqueness

A collection ID containing only alphanumerics and dots, starting and ending with an alphanumeric, and unique within the database is accepted. IDs containing other characters or duplicating an existing ID are rejected.

### AC: directory-shape

**Requirements:** collection#req:collection-directory, collection#req:schema-binding

A directory with a `.collection/definition.yaml` and records that conform to that schema is a valid collection. A directory missing the schema file, or containing records that violate the schema, fails validation.

## Outstanding Questions

- How are sub-collections discovered when the parent uses `record_file.type: map[string]any` (one file per record) versus the list/dictionary types where there is no per-record directory?

---
*This document follows the https://specscore.md/feature-specification*
