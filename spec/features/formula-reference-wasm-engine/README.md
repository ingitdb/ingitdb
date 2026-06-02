# Feature: Formula Reference WASM Engine

> [SpecScore.**Studio**](https://specscore.studio): | [Explore](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/formula-reference-wasm-engine?op=explore) | [Edit](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/formula-reference-wasm-engine?op=edit) | [Ask question](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/formula-reference-wasm-engine?op=ask) | [Request change](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/formula-reference-wasm-engine?op=request-change) |
**Status:** Draft
**Date:** 2026-06-02
**Owner:** alexander.trakhimenok@gmail.com
**Source Ideas:** computed-columns-and-scripted-views
**Supersedes:** —

## Summary

A reference Starlark engine compiled to WebAssembly that any inGitDB implementation
MAY embed to evaluate computed-column formulas, making byte-identical
cross-implementation results structural rather than tested-after-the-fact. This Feature
builds on the [`computed-columns`](../computed-columns/README.md) Feature, which defines
the formula language, deterministic sandbox, and the conformance vectors that remain the
single normative authority.

## Problem

Cross-platform implementations — a TypeScript client, the browser, Node — need
byte-identical formula evaluation. Reimplementing Starlark per language risks divergence,
especially in numeric and string corners. Shipping one sandboxed interpreter as a single
WASM module embedded in every host makes agreement structural: the same bytecode runs
everywhere. This Feature is **deferred** until cross-platform/browser execution is on the
roadmap; the Go reference implementation conforms today by passing the `computed-columns`
conformance vectors with its native Starlark engine, so no second implementation yet
exercises the cross-language agreement this Feature guarantees.

## Behavior

### The reference engine

#### REQ: reference-wasm-engine

Correctness is defined by the `computed-columns` conformance vectors, not by any
particular binary. To make cross-platform conformance easy, the standard PROVIDES a
**reference Starlark engine compiled to WebAssembly** that produces those vectors; an
implementation MAY embed this shared WASM engine to obtain conformance directly, or use
any other Starlark engine that passes every vector. The reference WASM engine is the
*recommended* path for new implementations (e.g. a TypeScript client) because it avoids
reimplementing the language; the Go reference implementation instead embeds a native
Starlark library and conforms by passing the same vectors. The reference engine is a
convenience and a tie-breaker for under-specified corners, never an authority above the
vectors.

### Cross-implementation agreement

#### REQ: structural-agreement

Embedding the identical WASM engine in every host makes byte-identical agreement
structural: given the same schema, record, and formula, two implementations embedding the
shared engine produce identical output by construction. An implementation that does not
embed the shared engine MUST still produce byte-identical output by passing every
`computed-columns` conformance vector.

## Acceptance Criteria

### AC: implementations-agree (verifies REQ:reference-wasm-engine, REQ:structural-agreement)

**Given** the same schema, record, and formula
**When** two conforming implementations each read the record
**Then** both produce byte-identical output for the computed column

## Rehearse Integration

No repo-local Rehearse stubs are generated. This is a Draft placeholder; acceptance is
verified by the cross-implementation conformance vectors defined in the `computed-columns`
Feature, run inside each implementation's own CI once a second implementation exists.

## Open Questions

- Which Starlark-to-WASM build is normative — `starlark-go` compiled to wasm, or `starlark-rust` — weighed on wasm binary size, evaluation performance, and browser startup cost?
- What are the acceptable wasm binary size and per-eval latency budgets on each host, including the browser?
- How does the Go reference implementation migrate from its native `starlark-go` embedding to the shared WASM engine, and on what timeline?

---
*This document follows the https://specscore.md/feature-specification*
