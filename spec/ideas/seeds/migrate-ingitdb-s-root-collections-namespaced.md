---
type: sidekick-seed
captured_by: specstudio:ideate
status: queued
---
# Migrate ingitdb's root-collections / namespaced implementation onto the shared standalone module system once SpecScore and DataTug have proven it

Spawned from DataTug's `shared-module-system` Idea (github.com/datatug/datatug-cli, spec/ideas/shared-module-system.md). That Idea proposes a standalone, product-agnostic module system (registry-by-path + namespacing + module-qualified references) in a dedicated repo (code name github.com/specscore/modules), proven first by SpecScore + DataTug. ingitdb already has the most mature namespaced implementation (`.ingitdb/root-collections.yaml`, namespace imports, dot-qualified refs) in `ingitdb-cli/pkg/ingitdb/config` — it is the extraction reference. This seed tracks migrating ingitdb onto the shared system **after** it is proven, so ingitdb stays aligned rather than diverging.

**Canonical reference syntax to adopt:** the shared system locks references as `module:item/path:field` — `:` separates module│item│field, `/` nests the item path, `.` is allowed inside module and item names, and `:` is forbidden inside segments (it stays fully syntactic and shell-safe, unlike a `#` field separator which breaks unquoted under zsh `extended_glob`). ingitdb today has **no** `#`/field-level reference syntax — it uses dotted collection ids (`todo.tasks`) plus collection-level `foreign_key: <collection-id>`. So this is not a literal `#`→`:` swap; it means: when ingitdb migrates, consider expressing its namespace/collection/field references in the colon-delimited canonical form (e.g. a field-level ref would become `todo.tasks:<record>:<column>`), so ingitdb and DataTug resolve identically. Owner: "probably OK" to adopt — confirm at migration time.
