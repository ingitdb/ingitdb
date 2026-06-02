# Feature: Repository Configuration

> [SpecScore.**Studio**](https://specscore.studio): | [Explore](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/repository-configuration?op=explore) | [Edit](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/repository-configuration?op=edit) | [Ask question](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/repository-configuration?op=ask) | [Request change](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/repository-configuration?op=request-change) |
**Status:** Draft

## Summary

A `.ingitdb/` directory at the repository root holds an inGitDB database's configuration: which collections exist, where they live, what languages are supported, and what default namespace applies. Every file inside `.ingitdb/` is optional — an absent or empty `.ingitdb/` directory is a valid empty inGitDB database.

## Problem

Every database needs a known place to declare its configuration. Scattering settings across the repo would force every tool to guess locations; embedding them in a single all-purpose file would conflate concerns. A dedicated `.ingitdb/` directory at the repo root with a small set of well-known files balances discoverability with separation of concerns.

## Behavior

### Directory location and shape

```
<repo-root>/
  .ingitdb/
    root-collections.yaml    # collection ID → directory path
    settings.yaml            # default_namespace, languages
    README.md                # human-readable overview (no code impact)
```

#### REQ: directory-at-root

The configuration directory MUST be located at `<repo-root>/.ingitdb/`. No other location is recognized.

#### REQ: all-files-optional

Every file inside `.ingitdb/` is OPTIONAL. A repository with no `.ingitdb/` directory, or with an empty `.ingitdb/`, MUST be treated as a valid empty inGitDB database.

### Recognized files

The directory has a fixed, well-known set of recognized files. Files not listed below are ignored by inGitDB tooling.

| File | Purpose |
|---|---|
| `root-collections.yaml` | Flat map of collection ID → directory path; supports namespace imports. |
| `settings.yaml` | Repository settings: `default_namespace`, `languages`. |
| `README.md` | Human-readable overview; documentation only, no code impact. |

#### REQ: known-file-set

inGitDB tooling MUST recognize `root-collections.yaml`, `settings.yaml`, and `README.md` inside `.ingitdb/`. Other files MUST be ignored, not error.

### Auto-detection

Tooling that runs from a sub-directory of a Git repository should walk upward to find `.ingitdb/` automatically.

#### REQ: auto-detection

When a tool is invoked from inside a Git repository, it MUST walk upward from the current working directory to find the nearest `.ingitdb/` directory.

## Dependencies

- root-collections
- default-namespace
- localization

## Acceptance Criteria

### AC: empty-config-is-valid

**Requirements:** repository-configuration#req:directory-at-root, repository-configuration#req:all-files-optional

A repository with no `.ingitdb/` directory, or one whose `.ingitdb/` directory is empty, is treated as a valid empty database with no collections.

### AC: well-known-files

**Requirements:** repository-configuration#req:known-file-set, repository-configuration#req:auto-detection

A repository with a `.ingitdb/root-collections.yaml` and `.ingitdb/settings.yaml` at the root is correctly detected from any sub-directory. Unknown files in `.ingitdb/` do not cause errors.

## Open Questions

- Should there be a per-user `.ingitdb/.local.yaml` ignored by Git for developer-specific overrides, or is configuration purely repo-level?

---
*This document follows the https://specscore.md/feature-specification*
