# Feature: Referential Integrity

**Status:** Draft

## Summary

A column may declare a `foreign_key` referencing another collection. Validation reports broken references — values that do not match any record ID in the referenced collection — as errors. The same metadata enables the materializer to generate FK-filtered views without a runtime query engine.

## Problem

Once data is split across collections, references between collections become inevitable: a `company` belongs to a `country`, a `task` belongs to a `status`. Without a declared foreign-key relationship, a typo in a referenced ID is silent and only manifests later when a downstream consumer fails to look it up. Declaring the relationship in the schema lets validation catch the typo at commit time and lets tooling generate efficient pre-joined output.

## Behavior

### Declaring a foreign key

A column in a collection schema may declare a `foreign_key` whose value is the ID of another collection.

```yaml
# companies/.collection/definition.yaml
columns:
  country:
    type: string
    foreign_key: countries
```

#### REQ: foreign-key-field

A column definition MAY include a `foreign_key` field whose value is a collection ID. The field is OPTIONAL.

#### REQ: foreign-key-target-must-exist

The collection ID named in `foreign_key` MUST resolve to an existing collection in the database (including via namespace imports). A `foreign_key` pointing at a non-existent collection is a configuration error.

### Validation

Validation checks every record's foreign-key column against the referenced collection's record IDs.

#### REQ: validate-references

For every record in a collection with a `foreign_key` column, validation MUST verify that the column's value matches an existing record ID in the referenced collection. A value that does not match MUST be reported as a validation error including the offending collection, record, and field.

#### REQ: empty-or-null-skipped

A `nil` or empty-string value in a `foreign_key` column MUST NOT be checked for existence. Whether such values are permitted is governed by general column type / required-ness, not by referential-integrity validation.

### Interaction with materialization

Foreign-key metadata is consumed by the materializer to generate per-FK-value filtered slices of the default view (FK-filtered views).

#### REQ: enables-fk-views

A collection that declares both a default view and at least one `foreign_key` column enables FK-filtered view generation for that column. The mechanics live in the [materialized-views](../materialized-views/README.md) feature.

## Dependencies

- collection-schema
- materialized-views

## Acceptance Criteria

Not defined yet.

## Outstanding Questions

- Acceptance criteria not yet defined for this feature.
- Should referential-integrity violations be configurable per-column as warning or error, or is the policy always strict?
- Are composite (multi-column) foreign keys in scope for this format, or does each foreign-key relationship live on a single column?
- How are foreign keys into namespace-imported collections expressed — by the local alias or by the imported database's own collection ID?

---
*This document follows the https://specscore.md/feature-specification*
