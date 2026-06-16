# PolyModel Parity Analysis — DataTug & inGitDB

Date: 2026-06-16
Status: Research / analysis

Compares the **PolyModel v0 core model** (as specified in
[`spec/features/core-model`](../features/core-model/README.md) and implemented in
`polymodel-go`) against two existing schema/model systems, to list what PolyModel is
**missing**:

- **DataTug** — semantic entities, table/view (dbmodel) definitions, and recordsets.
- **inGitDB** — the collection schema definition (`columns` / `ColumnDef`).

> Originally drafted for `spec/ideas/parity/`, but `spec/ideas/` is reserved for
> single-file SpecScore Idea artifacts, so this analysis lives under `spec/research/`.

### Scope rule

Not every feature belongs in PolyModel. PolyModel defines **data shape, constraints,
and relationships** — not storage layout, serialization, or UI. So:

- **Collection schema and constraints → in scope.**
- Record-file format, on-disk layout, README generation, conflict resolution, UI
  widgets → **out of scope** (these stay in the storage backend / tooling).

### Legend

| Mark | Meaning |
|---|---|
| ✅ | already in PolyModel v0 |
| ➖ | **missing — in scope** (should be added) |
| 🔶 | missing — borderline / needs a decision |
| 🚫 | missing — **out of scope** for PolyModel |

---

## PolyModel v0 baseline (what it already has)

- **Entity** — `property` (type, constraints, association to another entity, reuse of
  another property), embedded `component`s, `key`.
- **Collection** — `kind` (editable | computed), `field` (loose/optional, `bind` to an
  entity property, enforced `foreign_key`), `query` (opaque seam).
- **Recordset** — `column` (`bind`, `source` expression, soft nav `soft_ref`), `keys`,
  `query`.
- **Component** — reusable field group (Entities only).
- **Types** — `string, int, float, bool, decimal, uuid, date, time, datetime,
  document/json, any`; plus component embed and entity association.
- **Constraints** — `required, unique, min_len, max_len, pattern, enum`.

---

## A. vs inGitDB collection schema (`ColumnDef` / collection definition)

| inGitDB feature | PolyModel | Scope | Notes |
|---|---|---|---|
| Column `type` (primitive) | ✅ | — | PolyModel primitive set is a superset (adds decimal, uuid, document). |
| `required` | ✅ | — | |
| `length` (exact) | ➖ | in scope | PolyModel has min/max but no exact-length constraint. |
| `min_length` / `max_length` | ✅ | — | |
| `foreign_key` (column → collection) | ✅ | — | PolyModel collection `field.foreign_key`. |
| **Localized type** `map[locale]string` | ➖ | **in scope** | No localization in PolyModel. Big data-shape gap. |
| **Generic map type** `map[keyType]valueType` | ➖ | **in scope** | PolyModel has no map/dictionary type. |
| **`formula`** (Starlark computed column) | ➖ | **in scope** | PolyModel has no computed/stored-derived field (recordset `source` is result-only). |
| `format` hint (email, uri, markdown, html, pdf) | 🔶 | borderline | Semantic formats (email/uri) feel in-scope as constraints; rendering hints (markdown/html) are presentational. Split needed. |
| `title` / `titles` (localized labels) | ➖ | in scope | No human-facing labels on attributes (see Cross-cutting). |
| `valueTitle` | 🔶 | borderline | Label for a column's value; minor metadata. |
| `locale` (pairs a column with its localized map) | ➖ | in scope | Tied to localization adoption. |
| **`primary_key`** (collection-level) | ➖ | **in scope** | PolyModel Collections declare no key — only Entities have `key`. Collections need an identity. |
| `columns_order` | ✅ | — | Implicit in PolyModel (HCL declaration order). |
| `default_view` | 🔶 | borderline | Links a collection to a default view; more tooling than shape. |
| `data_dir` | 🚫 | out | Storage path. |
| `record_file` (name / format / type) | 🚫 | out | Serialization & on-disk layout. |
| `readme` (README generation) | 🚫 | out | Tooling. |
| `conflict_resolution` | 🚫 | out | Storage/merge strategy. |
| Collection `titles` (localized name) | ➖ | in scope | Object-level label (see Cross-cutting). |

PolyModel **already exceeds** inGitDB on: `unique`, `pattern`, `enum`, explicit
`editable`/`computed` kind, property reuse, components.

---

## B. vs DataTug entities (`*.entity.json`)

| DataTug feature | PolyModel | Scope | Notes |
|---|---|---|---|
| `fields` (id, type) | ✅ | — | PolyModel `property`. |
| `isKeyField` | ✅ | — | PolyModel Entity `key` list (different mechanism, same effect). |
| `title` (field & entity) | ➖ | in scope | Human-facing label. |
| **`namePatterns`** (auto-map entity field to columns by name) | 🔶 | borderline | Discovery/mapping aid; PolyModel models explicit `bind` instead. Auto-discovery may be tooling. |
| **`tables`** (entity → tables that contain it) | 🔶 | borderline | PolyModel models the binding from the *collection* side (`field.bind`), not the entity side. Reverse-index could be derived. |
| `tags` (categorization) | ➖ | in scope | Lightweight organizational metadata. |
| **`extends`** (URL to external / ISO entity) | ➖ | in scope (deferred) | PolyModel catalog / cross-module references (spec-maturity Gap 7) not yet built. |
| `description` (implied) | ➖ | in scope | No documentation field on objects/attributes. |

---

## C. vs DataTug recordsets & table/view (dbmodel) defs

| DataTug feature | PolyModel | Scope | Notes |
|---|---|---|---|
| Recordset `columns` (name, type) | ✅ | — | PolyModel `column`. |
| Column `meta` → `{entity, field}` | ✅ | — | PolyModel `column.bind = "Entity.property"`. |
| Column `dbType` (native db type) | 🔶 | borderline | Raw storage type alongside the semantic type; arguably a storage-projection concern. |
| `primaryKey` + `clustered` | 🔶 | partly | PolyModel recordset has `keys`; no `clustered` hint (storage-ish). |
| **`alternateKeys`** | ➖ | **in scope** | PolyModel has a single `key`/`keys`; no alternate keys (entities and recordsets). |
| **`foreignKeys`** (structured, on recordset) | 🔶 | borderline | PolyModel recordset has per-column `soft_ref` only; no structured FK list. |
| **`jsonSchema`** (shape of a JSON/document column) | ➖ | in scope | PolyModel has a `document` type but cannot describe its nested shape. |
| Query `parameters` | 🚫→DTQL | out (deferred) | Belongs to the query language (DTQL), behind PolyModel's opaque seam. |
| `hideIf` (conditional visibility) | 🚫 | out | UI/presentation. |
| `widgets` (lookup widgets) | 🚫 | out | UI/presentation. |
| Table `columns` (data_type, max_length, nullable) | ✅ | — | PolyModel field `type` + `optional` + `max_len`. |
| Views | ✅ | — | PolyModel `kind = "computed"`. |
| Table `primary_key` (clustered) / `foreign_keys` | 🔶 | partly | Collection-level primary key missing (see Table A); field FK present. |

---

## Cross-cutting in-scope gaps (appear in more than one system)

1. **Human-facing metadata** — `title` / `titles` (localized) / `description` on
   objects and attributes. Both DataTug (`title`) and inGitDB (`titles`) have it;
   PolyModel has none. (SpecScore entities also carry descriptions.)
2. **Localization** — localized field types (`map[locale]string`) and paired `locale`
   columns. inGitDB first-class; PolyModel absent.
3. **Map / dictionary types** — `map[keyType]valueType`. inGitDB has them; PolyModel
   has no container type beyond `document`.
4. **Collection-level identity** — a `primary_key` on Collections (PolyModel only keys
   Entities).
5. **Alternate keys** — both DataTug recordsets and relational tables; PolyModel has a
   single key.
6. **Computed/derived fields** — inGitDB `formula`; PolyModel has no stored computed
   field.
7. **Nested/document shape** — `jsonSchema` for a `document`-typed attribute.
8. **Exact-length constraint** — inGitDB `length`.
9. **Semantic formats** — `email` / `uri` (the in-scope half of inGitDB `format`).
10. **External references / catalog** — DataTug `extends`; PolyModel catalog (Gap 7),
    deferred.

## Explicitly out of scope (stays in the backend / tooling)

- inGitDB: `record_file` (name/format/type), `data_dir`, `readme`, `conflict_resolution`.
- DataTug: `hideIf`, `widgets`, query `parameters` (→ DTQL), and raw `dbType` /
  `clustered` storage hints (storage-projection concerns, not the canonical model).

## Suggested priority for in-scope gaps

| Tier | Gaps |
|---|---|
| **High** (core shape) | localization, map types, collection primary key, computed fields, nested document schema |
| **Medium** (richness) | metadata (titles/descriptions/tags), alternate keys, exact-length & semantic-format constraints |
| **Lower / deferred** | external references / catalog (Gap 7), name-pattern auto-mapping, native-type & clustered hints (borderline) |

Each High/Medium item is a candidate for a follow-on PolyModel Feature; several
(localization, map types, metadata) align with the deferred "type-system
reconciliation" decision already noted in the core-model spec.
