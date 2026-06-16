---
format: https://specscore.md/idea-specification
status: Specified
---

# Idea: PolyModel Core Model

**Status:** Specified
**Date:** 2026-06-16
**Owner:** alex
**Promotes To:** core-model
**Supersedes:** —
**Related Ideas:** —

## Problem Statement

How might we define a data model once — with the semantic model, the physical storage, and query results as distinct concepts — so one canonical model projects to many storage, transport, and language targets without drift?

## Context

Triggered by converging the portfolio (DataTug, SpecScore, OpenVaultDB, inGitDB) on PolyModel. Existing tools, notably DataTug, tangle the semantic model (what an Order IS), the physical schema (the table/view that stores it), and query results into one layer, so the model cannot be reused across targets. A mature design already exists at polymodel-org/polymodel-go/docs/v0-model.md and polymodel-org/backstage/concepts/model-layers.md.

## Recommended Direction

Decouple into three structural concepts — Entity (semantic, identity-bearing), Collection (a named, FROM-able data source with a Kind of Editable tables/FS-collections or Computed views; uniform loose fields, schemaless-capable), and Recordset (the tabular shape of a query or stored-procedure result). Give each layer its own attribute type named for what it is: entity.property (canonical/semantic), collection.field (loose, schemaless-capable), recordset.column (strict tabular). Add Component for reusable field groups. Queries (views, recordsets) use DTQL. The authored format is HCL, matching the spec; Go consumers use the in-memory model directly.

Four resolved design choices scope v0: (1) the DTQL query body sits behind a small placeholder seam in `polymodel-go` — v0 models view/recordset shapes now and swaps in the `dtql-go` AST once it stabilizes, with no hard dependency on the currently-empty repo; (2) Components embed into Entities only for v0; (3) the type system locks a minimal primitive set (`string`, `int`, `float`, `bool`, `decimal`, `uuid`, `date`, `time`, `datetime`, `document`/`json`, `any`) plus core constraints (`required`, `unique`, length, `pattern`, `enum`), deferring fuller reconciliation to a separate type-system decision; (4) at a binding site the local Collection field / Recordset column is primary and carries an optional reference to the Entity property it binds to — the collection/recordset owns its attribute and may override or add storage/result concerns.

## Alternatives Considered

- **One merged tabular attribute type for Collections and Recordsets.** Treat both as columnar with a shared type. Lost: a Collection can be schemaless or hold heterogeneous documents — "column" implies a fixed schema every record conforms to, which is wrong for document/Git/Firestore collections. A loose `field` and a strict `column` are genuinely different.
- **Separate `Table` and `View` types.** Model tables and views as distinct structural types. Lost: at `SELECT FROM` time a view is indistinguishable from a table, so a single `Collection` with a `Kind` attribute (Editable | Computed) is simpler and keeps the consumer contract uniform.
- **YAML/JSON-first authored format.** Author models in YAML/JSON since the consuming tools are YAML/JSON. Lost: the spec is HCL, and the Go consumers (inGitDB generator, SpecScore `@pm:` resolver) use the in-memory model directly rather than a serialized file, so serialization is not a v0 constraint.
- **A PolyModel-native query AST.** Define our own query language for views/recordsets. Lost: it duplicates DTQL, which already targets exactly this (dialect-agnostic query AST).

## MVP Scope

polymodel-go v0: the in-memory model types for Entity/property, Collection/field (with Kind), Recordset/column, and Component; HCL parsing into the model; intra-model reference resolution; and structural plus constraint validation with located errors. No target generators.

## Not Doing (and Why)

- Target generators (inGitDB, SQL, Firestore, language types) — separate downstream convergence efforts
- JSON/YAML serialization — HCL-first for v0; emit later only when a non-Go consumer needs it
- Catalog / remote-module resolution and @pm: references — a separate SpecScore-integration effort
- A PolyModel-native query language — reuse DTQL
- Migrations / schema evolution — out of scope for the core model

## Key Assumptions to Validate

| Tier | Assumption | How to validate |
|------|------------|-----------------|
| Must-be-true | The three-concept decoupling (Entity / Collection / Recordset) cleanly expresses real models from the portfolio. | Model 2–3 real schemas (e.g. DataTug northwind/chinook) in the v0 types and confirm nothing is forced. |
| Should-be-true | The placeholder query seam cleanly accepts the DTQL AST later without reshaping the model. | When dtql-go ships a stable AST, swap it in behind the seam and confirm Recordset/View shapes are unchanged. |
| Should-be-true | HCL-first authoring is acceptable because Go consumers use the in-memory model, not a serialized file. | Prototype the inGitDB schema generator against the in-memory model and confirm no serialization is needed. |
| Should-be-true | `property` / `field` / `column` naming reduces confusion versus a single shared term. | Review the three-attribute model against the portfolio tools' existing vocabularies. |
| Might-be-true | Components embed into Entities only (not Collections) for v0. | Revisit when modeling real schemas; promote to Collections only if a concrete need appears. |


## SpecScore Integration

- **New Features this would create:** a `core-model` Feature specifying the concepts and their rules; Entity/Property artifacts modeling Entity, Collection, Recordset, Component and their attributes; and four Decision artifacts for the forks (attribute model = property/field/column; no `Table` type / `Collection.Kind`; DTQL for queries; HCL-first authored format).
- **Existing Features affected:** none (this is the foundational model).
- **Dependencies:** DTQL (`dtql-go`, currently empty); HCL parsing (`hashicorp/hcl/v2`). Downstream convergence efforts (inGitDB generator, OpenVaultDB, SpecScore `@pm:` references, DataTug adoption) depend on this but are out of scope here.

## Open Questions

The four prior open questions are resolved and folded into the Recommended Direction (placeholder query seam; Components on Entities only; minimal type/constraint set with fuller reconciliation deferred; local field/column primary with an optional property reference). Each becomes a Decision artifact during Specify. The deferred type-system reconciliation is tracked there, not as an open question here.
