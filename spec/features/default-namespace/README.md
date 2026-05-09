# Feature: Default Namespace

**Status:** Draft

## Summary

The optional `default_namespace` field in `.ingitdb/settings.yaml` declares a namespace prefix that is automatically applied to a database's own collection IDs when the database is opened directly. When the same database is imported by another database via a namespace import, the import alias overrides `default_namespace`.

## Problem

A self-contained database often wants to namespace its own collections (e.g. `todo.tasks`, `todo.tags`) without forcing every consumer to repeat the prefix in `root-collections.yaml`. Declaring `default_namespace: todo` once lets the unprefixed `tasks: tasks` entry expand to `todo.tasks` automatically. But when another database imports this one as `productivity.*`, the importer's alias must win — otherwise a single database cannot be reused under different aliases.

## Behavior

### Field location

#### REQ: field-location

The `default_namespace` field MUST live in `.ingitdb/settings.yaml`. It is OPTIONAL — a missing field means no default namespace.

### Direct-open expansion

When a database is opened directly (not imported), every collection declared in its `root-collections.yaml` is presented with the `default_namespace` prefix.

```yaml
# demo-dbs/todo/.ingitdb/settings.yaml
default_namespace: todo
```

```yaml
# demo-dbs/todo/.ingitdb/root-collections.yaml
statuses: statuses
tags: tags
tasks: tasks
```

When opened directly, the collections are presented as `todo.statuses`, `todo.tags`, `todo.tasks`.

#### REQ: direct-open-prefix

When a database with a `default_namespace` is opened directly, every collection ID from its own `root-collections.yaml` MUST be prefixed with `<default_namespace>.`.

### Import-alias overrides default

When a database is imported by another database via a namespace import, the import alias replaces `default_namespace` for that import.

If `parent` imports `demo-dbs/todo` as `productivity.*`, the collections appear as `productivity.statuses`, `productivity.tags`, `productivity.tasks` — not `todo.*`.

#### REQ: import-alias-wins

When a database is consumed via a namespace import, the import alias MUST be used as the prefix and MUST override any `default_namespace` declared by the imported database.

### Charset

#### REQ: namespace-charset

The `default_namespace` value MUST follow the same charset rules as a collection ID component: alphanumerics and `.`, starting and ending with alphanumerics.

## Dependencies

- repository-configuration
- root-collections

## Acceptance Criteria

### AC: direct-open-uses-default

**Requirements:** default-namespace#req:field-location, default-namespace#req:direct-open-prefix, default-namespace#req:namespace-charset

A database whose `settings.yaml` sets `default_namespace: todo` and whose `root-collections.yaml` declares `tasks: tasks`, when opened directly, exposes the collection as `todo.tasks`.

### AC: import-overrides-default

**Requirements:** default-namespace#req:import-alias-wins

The same database, when imported by another database as `productivity.*`, exposes its `tasks` collection as `productivity.tasks`. The imported database's `default_namespace` does not appear in the resulting collection IDs.

## Outstanding Questions

- Can `default_namespace` itself contain a dot to declare a multi-segment namespace (e.g. `org.todo`), or is it constrained to a single segment?

---
*This document follows the https://specscore.md/feature-specification*
