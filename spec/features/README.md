# inGitDB Feature Index

This index lists every top-level feature that defines the inGitDB database format and its core principles. Features describe **what** the format and engine are — directory layout, schema rules, file types, and the role of Git as the storage engine. CLI behavior and REST API endpoints are specified in the `ingitdb-cli` and `ingitdb-server` repositories respectively.

## Index

| Feature | Status | Description |
|---|---|---|
| [storage-format](storage-format/README.md) | Draft | Records are plain YAML or JSON files on disk — human-readable and Git-diffable, with no proprietary binary encoding. |
| [collection](collection/README.md) | Draft | A collection is a directory of records validated against a single schema; collections may contain sub-collections. |
| [collection-schema](collection-schema/README.md) | Draft | `.collection/definition.yaml` declares columns, column types, and the record file layout for a collection. |
| [record-file-types](record-file-types/README.md) | Draft | Three supported record file types (`map[string]any`, `[]map[string]any`, `map[string]map[string]any`) determine how records are stored on disk. |
| [git-as-storage-engine](git-as-storage-engine/README.md) | Draft | Git is the storage engine — every change is a commit and branching, merging, bisect and revert work natively on data. |
| [repository-configuration](repository-configuration/README.md) | Draft | The `.ingitdb/` directory at the repository root holds optional configuration files; an absent or empty directory is a valid empty database. |
| [root-collections](root-collections/README.md) | Draft | `root-collections.yaml` is a flat map from collection ID to directory path, with namespace imports via the `prefix.*` syntax. |
| [default-namespace](default-namespace/README.md) | Draft | The optional `default_namespace` setting prefixes a database's own collection IDs when it is opened directly. |
| [localization](localization/README.md) | Draft | Multi-language support via the `languages` setting, declaring required and optional language codes. |
| [materialized-views](materialized-views/README.md) | Draft | Pre-computed view files generated under `$ingitdb/`, treated as derived artifacts and rebuilt during validation. |
| [referential-integrity](referential-integrity/README.md) | Draft | Foreign-key style references between collections; broken references are surfaced as validation errors. |

## Feature Summaries

### storage-format

inGitDB stores every record as a plain text file on disk in YAML or JSON. There is no proprietary binary format, no daemon, and no index file — a clone of the repository contains the entire database. Files are designed to be readable in any editor and produce useful, line-oriented diffs in any pull request.

### collection

A collection is a directory whose records all conform to a single schema. Each collection has a unique ID composed of alphanumerics and dots, and may contain sub-collections nested within individual records. Sub-collections enable naturally hierarchical data without joins.

### collection-schema

Every collection directory contains a `.collection/definition.yaml` file declaring its columns, their types, the record file layout, optional localized titles, and view configuration. The schema is the authoritative description of what records the collection may hold and is enforced during validation.

### record-file-types

The `record_file.type` field of a collection schema selects one of three storage layouts: one file per record (`map[string]any`), a single list file containing all records (`[]map[string]any`), or a single dictionary file keyed by record ID (`map[string]map[string]any`). Each type implies a distinct on-disk layout and Git diff behavior.

### git-as-storage-engine

Git is the engine, not an accessory. Every write produces a commit, every isolated change set is a branch, and the full history of the database is preserved. Standard Git operations — branch, merge, bisect, revert, blame, cherry-pick — apply directly to data records without translation.

### repository-configuration

A `.ingitdb/` directory at the repository root holds optional configuration: `root-collections.yaml`, `settings.yaml`, and an optional `README.md`. All files are optional; an empty or absent `.ingitdb/` directory is a valid empty inGitDB database.

### root-collections

`root-collections.yaml` is a flat YAML map from collection ID to a directory path. Namespace imports — entries whose key ends with `.*` — pull in all collections from another `root-collections.yaml`, prefixed with the import alias. Paths may be relative, absolute, or `~`-prefixed.

### default-namespace

The optional `default_namespace` field in `settings.yaml` prefixes a database's own collection IDs when the database is opened directly. When a database is imported via a namespace import (e.g. `todo.*`), the import alias overrides `default_namespace`.

### localization

The `languages` setting in `settings.yaml` lists the languages a database supports, marking each as `required` or `optional`. ISO 639-1 codes are preferred; IETF BCP 47 tags are also accepted. Required languages must appear before optional ones, and the declared order is the display order.

### materialized-views

Materialized views are pre-computed output files generated under the `$ingitdb/` directory at the repository root, mirroring the collection tree. They are treated as derived artifacts: rebuilt during validation, never edited by hand, and safe to delete because they can be regenerated.

### referential-integrity

Columns may declare a `foreign_key` referencing another collection. Validation reports broken references — values that do not match any record ID in the referenced collection. The same metadata enables the materializer to generate FK-filtered views without a query engine.

## Outstanding Questions

- How should sub-collections be discovered by the validator when they are nested inside record files of a `map[string]any` collection?
- Should referential-integrity validation be configurable (warn vs. error) per column, or is it always strict?

---
*This document follows the https://specscore.md/features-index-specification*
