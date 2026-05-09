# Feature: Root Collections

**Status:** Draft

## Summary

`.ingitdb/root-collections.yaml` is a flat YAML map declaring every root collection in a database. Each entry maps a collection ID to a directory path. Namespace imports — keys ending in `.*` — pull in all root collections from another database, prefixed with the import alias.

## Problem

A database needs an explicit, machine-readable list of where its collections live on disk. Scanning every directory for `.collection/definition.yaml` would be slow on large repositories and impossible over the GitHub REST API without expensive directory listings. A single flat map of collection ID → path makes lookup constant-time and lets remote tooling read the entire collection inventory in one HTTP call.

## Behavior

### Flat map format

The file is a single flat YAML map. No wrapper key, no nested structure.

```yaml
# .ingitdb/root-collections.yaml
companies: demo-dbs/test-db/companies
countries: demo-dbs/test-db/countries
```

#### REQ: flat-map

`root-collections.yaml` MUST be a flat YAML map. Each entry maps exactly one collection ID (key) to one collection directory path (value). Wrapper keys (`rootCollections:`, `collections:`, etc.) are not part of this format.

#### REQ: id-charset

Keys in `root-collections.yaml` MUST follow the collection ID rules: alphanumeric characters and `.` only, starting and ending with alphanumeric.

### Path values

Each value is a path to a single collection directory. Wildcards are not allowed in values.

#### REQ: single-directory-paths

The value of each entry MUST point to a single collection directory. Wildcard patterns such as `*` are not permitted in path values.

### Namespace imports

A key ending in `.*` is a namespace import. It pulls in every root collection from another `.ingitdb/root-collections.yaml`, prepending the import alias to each imported collection ID.

```yaml
todo.*: demo-dbs/todo
agile.*: demo-dbs/agile-ledger
```

If `demo-dbs/todo/.ingitdb/root-collections.yaml` declares `tasks: tasks`, the result is equivalent to `todo.tasks: demo-dbs/todo/tasks`.

#### REQ: namespace-import-syntax

A key ending in `.*` MUST be interpreted as a namespace import. The portion before `.*` is the import alias and is prepended to each imported collection ID.

#### REQ: namespace-import-errors

A namespace import MUST fail with an error when: the referenced directory does not exist, the referenced directory has no `.ingitdb/root-collections.yaml`, or the referenced `.ingitdb/root-collections.yaml` is empty.

### Path resolution

#### REQ: path-resolution

Path values MUST be resolved as follows:
- **Relative paths** are resolved relative to the directory containing the current `root-collections.yaml`.
- **Absolute paths** are used as-is.
- **`~`-prefixed paths** have `~` expanded to the user's home directory.

## Dependencies

- collection
- repository-configuration

## Acceptance Criteria

### AC: flat-map-parsing

**Requirements:** root-collections#req:flat-map, root-collections#req:id-charset, root-collections#req:single-directory-paths

A `root-collections.yaml` written as a flat map of valid collection IDs to single directory paths parses successfully and produces the expected list of collections.

### AC: namespace-import-expansion

**Requirements:** root-collections#req:namespace-import-syntax, root-collections#req:namespace-import-errors, root-collections#req:path-resolution

A namespace import like `todo.*: demo-dbs/todo` expands to `todo.<id>` for every collection in the imported database. Paths are resolved relative to the importing file's directory. Pointing a namespace import at a missing or empty target produces an error.

## Outstanding Questions

- Are recursive namespace imports (an imported database that itself has namespace imports) supported, and if so is there a depth limit?
- What happens when two namespace imports yield the same fully-qualified collection ID — error, or last-write-wins?

---
*This document follows the https://specscore.md/feature-specification*
