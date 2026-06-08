---
format: https://specscore.md/feature-specification
status: Approved
---

# Feature: Computed Columns

> [SpecScore.**Studio**](https://specscore.studio): | [Explore](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/computed-columns?op=explore) | [Edit](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/computed-columns?op=edit) | [Ask question](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/computed-columns?op=ask) | [Request change](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/computed-columns?op=request-change) |
**Status:** Approved
**Date:** 2026-06-02
**Owner:** alexander.trakhimenok@gmail.com
**Source Ideas:** computed-columns-and-scripted-views
**Supersedes:** —

## Summary

A collection column may declare an inline **formula** whose value is derived from the
record's other stored fields rather than stored on disk. This Feature defines computed
columns normatively — the formula language, evaluation semantics, typing, and
error behavior — so that every inGitDB implementation (the Go CLI, a TypeScript
client, and future ones) produces **byte-identical** results from the same schema and
data.

## Problem

Consumers routinely need derived values — a full name from two fields, a label, a
total — that stay in sync with their source fields. Without a normative definition,
each implementation would invent its own formula language and semantics, and the same
schema would yield different values across clients, breaking the "one database, many
implementations" promise. Computed columns must be specified once, at the standard
level, with cross-implementation agreement guaranteed.

## Behavior

### Declaring a computed column

#### REQ: computed-column-formula

A column MAY declare a `formula` string. A column whose `formula` is non-empty is a
*computed column*: its value is derived from the record's other fields and is never
read from, nor written to, the record's stored data.

#### REQ: formula-is-single-expression

A `formula` MUST be a single expression in the inGitDB formula language. A schema
whose computed column carries a formula that is not a single valid expression MUST be
rejected by validation, with an error naming the collection and the column.

#### REQ: formula-references-stored-fields-only

A formula MAY reference only stored (non-computed) sibling fields of the same record.
A formula that references another computed column MUST be rejected by validation,
with an error stating that a formula may reference only stored fields.

#### REQ: computed-column-types

A computed column MUST declare one of the scalar types `string`, `int`, `float`, or
`bool`. A computed column declared with any other type MUST be rejected by validation.

### The formula language

#### REQ: formula-language-starlark

The inGitDB formula language is **Starlark**: a formula is a single Starlark
expression, evaluated under Starlark's specified semantics. Starlark's specification
defines the *intent* of every formula; where Starlark leaves a corner under-specified,
the published conformance vectors (REQ:conformance-vectors) are the tie-breaker.

A reference Starlark engine compiled to WebAssembly — a convenience that lets new
implementations (e.g. a TypeScript client) obtain conformance without reimplementing the
language — is specified separately in the
[`formula-reference-wasm-engine`](../formula-reference-wasm-engine/README.md) Feature and is
subordinate to the conformance vectors. An implementation conforms by passing the vectors
with any Starlark engine; the Go reference implementation embeds a native Starlark library.

#### REQ: deterministic-sandbox

Formula evaluation MUST be deterministic and side-effect-free: it MUST NOT access the
network, filesystem, clock, or any source of randomness. Given the same record and
formula, every conforming implementation MUST produce identical output.

### Evaluation and typing

#### REQ: compute-on-read

A computed column's value MUST be produced when a record is read, from the record's
stored fields. It MUST NOT be persisted. A write (create or update) that supplies a
value for a computed column MUST be rejected, with an error naming the collection,
record key, and column.

#### REQ: type-coercion

The evaluated result MUST be coerced to the column's declared type. A result that
cannot be represented in the declared type MUST cause a fail-loud error
(REQ:fail-loud-on-error).

#### REQ: numeric-model

An `int` column holds a 64-bit signed integer; a `float` column holds an IEEE-754
double. Starlark integer division (`//`) and float division (`/`) apply as specified.
A non-integral value coerced into an `int` column, or an integer result outside the
64-bit signed range, MUST cause a fail-loud error rather than silent truncation or
wrap-around.

#### REQ: string-model

Strings are sequences of Unicode code points encoded as **UTF-8**, with no normalization
applied. Conversion of a non-string result into a `string`-typed computed column MUST be
deterministic and locale-independent, following Starlark's `str()` semantics: an integer
renders in base-10 with no separators or sign for non-negative values; a float renders in
Starlark's canonical decimal form; a boolean renders as `True`/`False`. No locale,
precision, or formatting setting may alter the byte sequence produced.

### Error handling

#### REQ: fail-loud-on-error

If a formula raises an evaluation error, or its result cannot be coerced to the
declared type, the read MUST fail with an error naming the collection, record key, and
column. An implementation MUST NOT return a partial record or a silently-empty
computed value in that case.

### Cross-implementation conformance

#### REQ: conformance-vectors

Conformance is defined **solely** by the vectors. The standard publishes **conformance
vectors** — tuples of (declared column type, formula, input stored fields, expected
output *or* expected error). An implementation conforms to computed columns **if and
only if** it produces the expected result for every published vector. The vectors —
not the language reference and not any particular embedding library or engine artifact —
are the single normative authority; the Starlark specification states intent, and any
reference engine (see the `formula-reference-wasm-engine` Feature) is a convenience, both
subordinate to the vectors.

## Acceptance Criteria

### AC: full-name-computed (verifies REQ:computed-column-formula, REQ:compute-on-read, REQ:formula-language-starlark)

**Given** a schema with a `string` column `full_name` whose formula is `first_name + " " + last_name`, and a record with `first_name: "Ada"`, `last_name: "Lovelace"`
**When** a conforming implementation reads the record
**Then** the record's `full_name` is `"Ada Lovelace"`, and the stored record file contains no `full_name` field

### AC: malformed-formula-rejected (verifies REQ:formula-is-single-expression)

**Given** a computed column whose formula is `first_name +` (not a complete expression)
**When** the schema is validated
**Then** validation fails with an error naming the collection and the column

### AC: chained-reference-rejected (verifies REQ:formula-references-stored-fields-only)

**Given** a computed column `greeting` whose formula references another computed column `full_name`
**When** the schema is validated
**Then** validation fails with an error stating that a formula may reference only stored fields

### AC: unsupported-type-rejected (verifies REQ:computed-column-types)

**Given** a computed column declared with a type other than `string`, `int`, `float`, or `bool`
**When** the schema is validated
**Then** validation fails naming the collection and the column

### AC: no-io-access (verifies REQ:deterministic-sandbox)

**Given** a formula that attempts to access the network, filesystem, clock, or randomness
**When** the formula is evaluated
**Then** evaluation fails because those capabilities are absent from the sandbox

### AC: string-output-deterministic (verifies REQ:string-model)

**Given** a `string` column whose formula renders a number, e.g. `str(qty)` with `qty: 3`
**When** two conforming implementations read the record
**Then** both produce the byte-identical UTF-8 string `"3"`, with no locale-dependent formatting

### AC: int-coercion (verifies REQ:type-coercion, REQ:numeric-model)

**Given** an `int` column `total` whose formula is `qty * price`, and a record with `qty: 3`, `price: 4`
**When** the record is read
**Then** `total` is the integer `12`

### AC: non-integral-into-int-fails (verifies REQ:numeric-model, REQ:fail-loud-on-error)

**Given** an `int` column whose formula is `7 / 2` (float division yielding `3.5`)
**When** the record is read
**Then** the read fails with an error naming the column, rather than truncating to `3`

### AC: reject-stored-computed-value (verifies REQ:compute-on-read)

**Given** a write that supplies a value for a computed column
**When** the write is attempted
**Then** it fails with an error naming the collection, record key, and column

### AC: runtime-error-fails-read (verifies REQ:fail-loud-on-error)

**Given** an `int` computed column whose formula divides by a field that is zero for some record
**When** that record is read
**Then** the read fails with an error naming the collection, record key, and column, and no partial record is returned

### AC: conformance-vectors-pass (verifies REQ:conformance-vectors)

**Given** the published computed-column conformance vectors
**When** a conforming implementation evaluates every vector
**Then** it produces the expected output, or the expected error, for each one

## Rehearse Integration

No repo-local Rehearse stubs are generated. This is a normative specification repo with
no implementation to exercise; acceptance is verified by the cross-implementation
**conformance vectors** (REQ:conformance-vectors), run inside each implementation's own
CI (e.g. the Go CLI and a TypeScript client), not by Rehearse scenarios in this repo.

## Open Questions

- What is the exact curated set of deterministic builtin helpers a formula may call (string methods, `len`/`min`/`max`, numeric helpers), and which Starlark universe members are excluded to preserve determinism?
- In what format and location are the conformance vectors published and versioned so every implementation's CI consumes the same set?
- Should computed columns be allowed to declare a `foreign_key` (validated against the derived value), extending `referential-integrity`? Deferred from this MVP.

---
*This document follows the https://specscore.md/feature-specification*
