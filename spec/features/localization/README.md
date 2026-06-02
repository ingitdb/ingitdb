# Feature: Localization

> [SpecScore.**Studio**](https://specscore.studio): | [Explore](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/localization?op=explore) | [Edit](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/localization?op=edit) | [Ask question](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/localization?op=ask) | [Request change](https://specscore.studio/app/github.com/ingitdb/ingitdb/spec/features/localization?op=request-change) |
**Status:** Draft

## Summary

inGitDB supports multi-language record fields through the `languages` setting in `.ingitdb/settings.yaml`. Each entry declares a language code as either `required` or `optional`. Required languages must appear before optional ones, and the declared order is the display order used by tooling.

## Problem

Reference data — country names, product titles, status labels — frequently needs translations. A schema-level concept of "this field has values in N languages" is cleaner than encoding the language as part of the field name (`name_en`, `name_fr`, `name_es`). Centralizing the supported language list in repo configuration also lets validation flag missing required translations.

## Behavior

### `languages` setting

`languages` is a list of single-key maps, where each entry's key is `required` or `optional` and the value is a language code.

```yaml
# .ingitdb/settings.yaml
languages:
  - required: en
  - required: fr
  - required: es
  - optional: ru
```

#### REQ: setting-location

The `languages` setting MUST live in `.ingitdb/settings.yaml`. It is OPTIONAL — a database without `languages` is treated as not localized.

#### REQ: list-of-required-or-optional

Each entry in the list MUST be a single-key map whose key is either `required` or `optional` and whose value is a language code.

### Language codes

ISO 639-1 codes are preferred; IETF BCP 47 (RFC 5646 / RFC 4647) tags are also accepted, allowing region-specific variants like `es-ES` and `es-MX`.

#### REQ: code-format

A language code MUST be either an ISO 639-1 two-letter code or a valid IETF BCP 47 language tag. ISO 639-1 is the preferred form.

### Order rules

#### REQ: required-before-optional

In the `languages` list, every `required` entry MUST appear before any `optional` entry. Mixing them is a configuration error.

#### REQ: declared-order-is-display-order

The order in which languages appear in `languages` MUST be treated as the display order by tooling that needs to render a language selector or iterate language variants.

## Dependencies

- repository-configuration

## Acceptance Criteria

### AC: language-list-parsing

**Requirements:** localization#req:setting-location, localization#req:list-of-required-or-optional, localization#req:code-format

A `languages` list of `required:` and `optional:` entries with valid ISO 639-1 or BCP 47 codes parses successfully and yields the declared language set with required/optional flags preserved.

### AC: required-before-optional-enforced

**Requirements:** localization#req:required-before-optional, localization#req:declared-order-is-display-order

A configuration with required languages first followed by optional languages is accepted; one with optional languages preceding required ones is rejected. The declared order is preserved end-to-end.

## Open Questions

- How does a localized field appear in a record file — as a map keyed by language code, or via a field-name suffix convention?
- Should validation flag a record that is missing a translation in a required language as an error or as a warning?

---
*This document follows the https://specscore.md/feature-specification*
