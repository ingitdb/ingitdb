# Computed-Columns Conformance Vectors

This directory publishes the **normative conformance vectors** for the
[`computed-columns`](../../spec/features/computed-columns/README.md) Feature. An inGitDB
implementation conforms to computed columns **if and only if** it produces the expected
output, or the expected error, for every vector in [`vectors.yaml`](./vectors.yaml).

The vectors — not any prose, library, or engine — are the single normative authority
(see `REQ:conformance-vectors`).

## Execution profile (normative)

A formula is a single [Starlark](https://github.com/bazelbuild/starlark) expression
evaluated in a deterministic, side-effect-free sandbox:

- **Allowed:** the Starlark *universe* — which is deterministic and contains no I/O,
  clock, or randomness by construction — including `len`, `min`, `max`, `str`, `int`,
  `float`, `bool`, `range`, `sorted`, `reversed`, `enumerate`, and the native string
  methods. `str` is explicitly in the allowed set (it backs the string model).
- **Plus curated numeric helpers:** `abs`, `round`, `floor`, `ceil`. `round`/`floor`/`ceil`
  return an `int`; `abs` preserves `int` vs `float`.
- **Excluded:** any capability that would touch the network, filesystem, clock, or a
  source of randomness, and `load()`. `print()` is a no-op (no side effect). Evaluation
  is bounded by a step ceiling to bound abuse.

This profile resolves the source Feature's builtin-set Open Question.

## Numeric and string model (normative)

- `int` is a 64-bit signed integer; `float` is an IEEE-754 double.
- Starlark integer division (`//`) and float division (`/`) apply as specified. `7 // 2`
  is `3`; `7 / 2` is `3.5`.
- Coercing a non-integral `float` into an `int` column, or an integer result outside the
  signed 64-bit range, is a **fail-loud** error — never silent truncation or wrap-around.
- Strings are UTF-8 with no normalization. `str()` is locale-independent: an integer
  renders in base-10 (`str(3)` → `"3"`); a float in Starlark's canonical decimal form; a
  boolean as `True`/`False`.

## Vector format

`vectors.yaml` is a versioned list of evaluation vectors. Each vector is a tuple of
`(declared column type, formula, input stored fields, expected output OR expected error
kind)`:

```yaml
version: 1
vectors:
  - name: <unique-kebab-case-id>
    column_type: string | int | float | bool
    formula: <single Starlark expression>
    fields:                 # stored sibling fields bound as formula variables
      <field>: <scalar>
    expect: <expected coerced value>          # exactly one of expect / expect_error
    expect_error: <error-kind>                # see "Error kinds" below
```

- Exactly one of `expect` or `expect_error` MUST be present.
- `expect` is the value after coercion to `column_type`: a string, a 64-bit integer, an
  IEEE-754 float, or a boolean.
- Errors are compared by **kind**, not by message text — an implementation's prose may
  differ, but the failure MUST occur and MUST name the offending column.

### Error kinds

| Kind | Meaning |
|---|---|
| `non-integral-coercion` | a non-integral `float` result was coerced into an `int` column |
| `unsupported-result-type` | the result's type cannot be coerced to the declared `column_type` |
| `runtime-error` | the formula raised during evaluation (e.g. division by zero) |
| `undefined-name` | the formula referenced a name absent from the sandbox (e.g. an I/O builtin like `open`) |

## Open Questions

- Versioning/distribution: this MVP versions vectors in-tree alongside the standard
  (the `version:` field pins the set). A shared test-vectors package may follow.

---
*This document follows the https://specscore.md/index-specification*
