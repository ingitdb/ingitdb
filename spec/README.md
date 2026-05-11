# inGitDB Specification

This directory holds the normative specification for the **inGitDB database format and engine** — the contract that every implementation (CLI, server, language clients) must honor.

The spec is written in [SpecScore](https://specscore.md/) format: features are atomic, each declares its requirements (REQs) and acceptance criteria (ACs), and references between features are explicit.

## Structure

| Path                                                                     | Purpose                                                            |
| ------------------------------------------------------------------------ | ------------------------------------------------------------------ |
| [`features/`](./features/README.md)                                      | Top-level features that define the format, schema, and engine.     |

Each feature lives in its own directory under `features/<feature-name>/README.md` and follows the [SpecScore feature specification](https://specscore.md/feature-specification). The features index at [`features/README.md`](./features/README.md) lists every feature with its current status and a one-sentence summary.

## Scope

The spec describes **what** inGitDB is — directory layout, schema rules, record file types, storage format, the role of Git as the storage engine.

It does **not** describe:

- CLI command-line behavior — see the [`ingitdb-cli`](https://github.com/ingitdb/ingitdb-cli) repository.
- REST API endpoints — see [`docs/open-api/`](../docs/open-api/) and the `ingitdb-server` repository.
- Client SDK design — see [`docs/ingitdb-ts-architecture.md`](../docs/ingitdb-ts-architecture.md).

## Validation

The spec tree is linted against SpecScore structural conventions using:

```bash
specscore spec lint --severity info
```

A green lint run is a precondition for any change to this directory.

## Related

- [Ideation history](../docs/ideas/) — design conversations that produced specs.
- [JSON Schemas](https://github.com/ingitdb/ingitdb-schema) — machine-readable schemas derived from this spec.
