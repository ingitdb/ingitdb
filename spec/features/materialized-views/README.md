# Feature: Materialized Views

> [SpecScore.**Studio**](https://specscore.studio): | [Explore](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/materialized-views?op=explore) | [Edit](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/materialized-views?op=edit) | [Ask question](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/materialized-views?op=ask) | [Request change](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/materialized-views?op=request-change) |
**Status:** Draft

## Summary

Materialized views are pre-computed output files generated under the `$ingitdb/` directory at the repository root, mirroring the collection tree. They include collection-level views, record-level views, and root-level views, and are treated as derived artifacts: rebuilt during validation, never edited by hand, and safe to delete because they can be regenerated.

## Problem

Web apps and AI agents often need shaped, filtered, or joined views of the data — "top 100 cities by population", "all companies in country X", "languages spoken in this city's region". Computing these on every read requires a query engine; pre-computing them as static files lets a consumer fetch the answer with a single HTTP request and no engine at all. The trade-off is that the views must be regenerated when the source data changes, which inGitDB ties to validation.

## Behavior

### View types

There are three view types, distinguished by what they take as input.

| Type | Input | Output |
|---|---|---|
| Collection-level | A single collection's records | Filter / lookup / render output for that collection |
| Record-level | A single record | A child view stored alongside the record |
| Root-level | Multiple collections | A joined output that draws from many collections |

#### REQ: three-view-types

inGitDB MUST support collection-level, record-level, and root-level views as the three view categories.

### Output location

All views are materialized to the `$ingitdb/` directory at the repository root, mirroring the collection directory tree. There is no per-collection `$views/` subfolder.

```
<repo-root>/
  $ingitdb/
    countries/
      $fk/
        companies/
          country/
            gb.ingr
    cities/
      top-100.json
```

#### REQ: output-under-$ingitdb

Materialized view output MUST be written under `<repo-root>/$ingitdb/`. The directory tree under `$ingitdb/` MUST mirror the collection directory tree.

#### REQ: no-collection-local-views-dir

Views MUST NOT be written into a `$views/` subdirectory inside individual collection directories. The single `$ingitdb/` tree at the repository root is the only output location.

### Derived-artifact semantics

Views are not source data. They are regenerated from the source collections.

#### REQ: derived-artifact

Materialized views MUST be treated as derived artifacts. Tooling MAY delete and rebuild them at any time. Hand-edited changes to view files are not preserved across rebuilds.

#### REQ: rebuild-on-validate

Validation passes (the database-walking pipeline that re-checks records against schemas) MUST rebuild materialized views in the same pass. After a successful validation, the view tree under `$ingitdb/` reflects the current source data.

### Idempotency

#### REQ: idempotent-output

Rebuilding views from unchanged source data MUST produce byte-identical output files. A second rebuild over the same data is a no-op as far as Git is concerned.

## Dependencies

- collection
- collection-schema
- referential-integrity

## Acceptance Criteria

### AC: views-emitted-under-$ingitdb

**Requirements:** materialized-views#req:output-under-$ingitdb, materialized-views#req:no-collection-local-views-dir

After a validation pass, every materialized view file lives under `<repo-root>/$ingitdb/` with a path that mirrors the source collection's directory. No view files appear inside individual collection directories.

### AC: rebuild-is-idempotent

**Requirements:** materialized-views#req:rebuild-on-validate, materialized-views#req:idempotent-output, materialized-views#req:derived-artifact

Running validation twice in a row over unchanged source data produces no Git diff in the `$ingitdb/` tree on the second run. Deleting the `$ingitdb/` tree and re-running validation regenerates it identically.

## Open Questions

- Are stale view files (corresponding to records that have been deleted) cleaned up automatically, or only on a full rebuild?
- Should views be excluded from `git diff` summaries by default to reduce review noise on large data changes?

---
*This document follows the https://specscore.md/feature-specification*
