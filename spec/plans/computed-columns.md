# Plan: Computed Columns

**Status:** Approved
**Source Feature:** computed-columns
**Date:** 2026-06-02
**Owner:** alexander.trakhimenok@gmail.com
**Supersedes:** —

## Summary

Decomposes the normative `computed-columns` Feature into the standardization
deliverables that make it conformance-checkable: the pinned deterministic execution
profile, the declaration/validation rules and vector format, the conformance vectors
themselves, and a proof that the Go CLI reference implementation passes every vector. The
reference Starlark→WASM engine and cross-implementation agreement are out of scope — they
moved to the dedicated `formula-reference-wasm-engine` Feature.

## Approach

Tasks are ordered so each one's output feeds the next: Task 1 pins the deterministic
builtin set and no-I/O sandbox; Task 2 fixes the vector file format and the
schema-validation rules for declaring computed columns; Task 3 authors the vectors in
that format against that profile; Task 4 proves the Go CLI reference implementation
produces the expected output or error for every vector. The two actionable Outstanding
Questions this Feature still carries (builtin set, vector format/location) are resolved
inside Tasks 1 and 2. The reference WASM engine and the `implementations-agree`
cross-implementation proof are not in this Plan — they belong to the deferred
`formula-reference-wasm-engine` Feature and require a second independent implementation.
The deferred computed-column foreign-key question stays an Outstanding Question (it names
no AC) and is intentionally out of this Plan.

## Tasks

### Task 1: Pin the deterministic execution profile

**Verifies:** computed-columns#ac:no-io-access

Enumerate the curated deterministic builtin/helper set a formula may call and the Starlark
universe members excluded to preserve determinism — the absence of network, filesystem,
clock, and randomness capabilities that makes a no-I/O formula fail. Resolves the
builtin-set Outstanding Question.

### Task 2: Specify the vector format and computed-column declaration rules

**Verifies:** computed-columns#ac:malformed-formula-rejected, computed-columns#ac:chained-reference-rejected, computed-columns#ac:unsupported-type-rejected, computed-columns#ac:reject-stored-computed-value

Define the conformance-vector file format — the (declared type, formula, input fields,
expected output *or* expected error *tag*) tuple shape, with errors compared by structured
kind rather than message text — and where vectors are published and versioned so every
implementation's CI consumes the same set. In the same pass, pin the normative
schema-validation rules for declaring a computed column: rejecting a non-single-expression
formula, a formula that references another computed column, an unsupported declared type,
and a write that supplies a value for a computed column. Resolves the vector-format/location
Outstanding Question.

### Task 3: Author the conformance vectors

**Verifies:** computed-columns#ac:full-name-computed, computed-columns#ac:int-coercion, computed-columns#ac:string-output-deterministic, computed-columns#ac:non-integral-into-int-fails, computed-columns#ac:runtime-error-fails-read

Author the cross-implementation vector set in the Task 2 format against the Task 1
profile, covering evaluation (the `full_name` canonical case), integer coercion, the
locale-independent string model, fail-loud non-integral-into-`int` coercion, and
fail-loud runtime errors — each with its expected output or expected error tag.

### Task 4: Prove the vectors against the Go CLI reference implementation

**Verifies:** computed-columns#ac:conformance-vectors-pass

Run every vector authored in Task 3 through the Go CLI's native Starlark formula
evaluator and confirm it produces the expected output or expected error for each one,
establishing that a conforming implementation passes every published vector. (Proving a
*second* implementation agrees byte-for-byte is the dedicated
`formula-reference-wasm-engine` Feature's job, not this Plan's.)

## Open Questions

- Should computed columns be allowed to declare a `foreign_key` validated against the derived value (extending `referential-integrity`)? Deferred from this MVP; tracked as an Outstanding Question on the source Feature, names no AC, and is out of this Plan.

---
*This document follows the https://specscore.md/plan-specification*
