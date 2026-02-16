# inGitDB JSON Schemas

This directory contains [JSON Schema](https://json-schema.org/) definitions for **inGitDB** configuration and collection definition files.

## Schemas

### `ingitdb-root-config.schema.json`

Schema for the **`.ingitdb.yaml`** root configuration file. This file lives at the root of an inGitDB database repository and maps collection IDs to their directory paths.

**Key properties:**

| Property          | Type                | Description                                                                                     |
| ----------------- | ------------------- | ----------------------------------------------------------------------------------------------- |
| `rootCollections` | `map[string]string` | Maps collection IDs to relative directory paths. A trailing `*` auto-discovers sub-collections. |

### `ingitdb-collection.schema.json`

Schema for **`.ingitdb-collection.yaml`** files. Each collection directory contains one of these files to define its structure.

**Top-level properties:**

| Property        | Type                   | Required | Description                                           |
| --------------- | ---------------------- | -------- | ----------------------------------------------------- |
| `titles`        | `map[string]string`    | No       | Localized collection titles keyed by locale code.     |
| `record_file`   | `RecordFileDef`        | **Yes**  | Defines file naming, format, and record storage type. |
| `data_dir`      | `string`               | No       | Relative path to the data files directory.            |
| `columns`       | `map[string]ColumnDef` | **Yes**  | Column definitions (at least one required).           |
| `columns_order` | `string[]`             | No       | Display order of columns (unique, no duplicates).     |
| `default_view`  | `string`               | No       | ID of the default view.                               |

**`RecordFileDef`** — defines how records are stored:

| Property | Type     | Required | Description                                                         |
| -------- | -------- | -------- | ------------------------------------------------------------------- |
| `name`   | `string` | **Yes**  | File name pattern (supports `{key}` and column placeholders).       |
| `format` | `string` | **Yes**  | Serialization format (e.g. `JSON`, `YAML`).                         |
| `type`   | `enum`   | **Yes**  | `map[string]any` · `[]map[string]any` · `map[string]map[string]any` |

**`ColumnDef`** — defines a column in the collection:

| Property      | Type                | Required | Description                                                                                                                      |
| ------------- | ------------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `type`        | `string` / `enum`   | **Yes**  | Data type: `string`, `int`, `float`, `bool`, `date`, `time`, `datetime`, `any`, `map[locale]string`, or `map[keyType]valueType`. |
| `title`       | `string`            | No       | Single-locale display title.                                                                                                     |
| `titles`      | `map[string]string` | No       | Localized titles keyed by locale.                                                                                                |
| `valueTitle`  | `string`            | No       | Display title for the column value.                                                                                              |
| `required`    | `boolean`           | No       | Whether a value is required (default: `false`).                                                                                  |
| `length`      | `integer`           | No       | Exact required length.                                                                                                           |
| `min_length`  | `integer`           | No       | Minimum length.                                                                                                                  |
| `max_length`  | `integer`           | No       | Maximum length.                                                                                                                  |
| `foreign_key` | `string`            | No       | Reference to another collection (e.g. `auth.users`).                                                                             |

## Usage

Reference the schemas in your YAML files for IDE validation and autocompletion:

```yaml
# yaml-language-server: $schema=./schemas/ingitdb-collection.schema.json
```

## Source of Truth

These schemas are derived from the Go struct definitions in [`ingitdb-go/ingitdb/`](../../ingitdb-go/ingitdb/):

- [`definition.go`](../../ingitdb-go/ingitdb/definition.go) — `Definition`
- [`collection_def.go`](../../ingitdb-go/ingitdb/collection_def.go) — `CollectionDef`
- [`record_file_def.go`](../../ingitdb-go/ingitdb/record_file_def.go) — `RecordFileDef`
- [`column_def.go`](../../ingitdb-go/ingitdb/column_def.go) — `ColumnDef`
- [`column_type.go`](../../ingitdb-go/ingitdb/column_type.go) — `ColumnType`
- [`config/root_config.go`](../../ingitdb-go/ingitdb/config/root_config.go) — `RootConfig`
