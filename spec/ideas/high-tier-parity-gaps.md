---
format: https://specscore.md/idea-specification
status: Draft
---

# Idea: Core model parity: high-tier gaps

**Status:** Draft
**Date:** 2026-06-16
**Owner:** alex
**Promotes To:** —
**Supersedes:** —
**Related Ideas:** —

## Problem Statement

How might we close the highest-impact data-shape gaps between PolyModel and DataTug/inGitDB — localization, map types, collection keys, computed fields, and nested document schema — so PolyModel can losslessly represent their schemas?

## Context

The parity analysis (spec/research/parity-datatug-ingitdb.md) found five high-tier, in-scope features that PolyModel v0 cannot express, while DataTug and/or inGitDB can. These are pure data-shape gaps (not storage or UI), and several align with the type-system reconciliation deferred in the core-model spec and decisions.

## Recommended Direction

Extend the core model with the five high-tier gaps: (1) localized field types (map[locale]string) plus the paired locale concept; (2) generic map/dictionary types (map[keyType]valueType); (3) a collection-level primary key (today only Entities have a key); (4) computed/derived fields (the inGitDB formula equivalent), distinct from a recordset column's source expression; (5) a nested schema for document/json-typed attributes (the DataTug jsonSchema equivalent). Each lands as a focused addition to the model types, HCL grammar, and validation, with polymodel-go updated in lockstep.

## Alternatives Considered

- **Do all parity gaps at once (high + medium + lower).** One big model expansion
  covering every gap from the analysis. Lost: too large to land cleanly; the medium
  and lower tiers are lower-impact and some are borderline-scope, so bundling them
  dilutes focus and delays the high-impact wins.
- **Treat each gap as its own independent Idea/Feature.** Five separate tracks. Lost:
  they share design surface (the type system, HCL grammar, validation) and several are
  type-system changes that should be reconciled together; five parallel Ideas would
  re-litigate the same type-system questions five times.
- **Defer until a real consumer demands it.** Wait for the inGitDB generator or DataTug
  adoption to force each gap. Lost: the parity analysis already establishes the need
  with concrete sources; closing the high-tier gaps now de-risks every downstream
  convergence effort (generation can't be lossless over a model that can't represent
  the source).

## MVP Scope

The five high-tier features above, added to the in-memory model, HCL parsing, and validation, with tests and a polymodel-go release. No generators.

## Not Doing (and Why)

- Medium-tier gaps (titles/descriptions/tags metadata, alternate keys, exact-length and semantic-format constraints) — a separate follow-on
- Lower/deferred gaps (external references/catalog, name-pattern auto-mapping, native-type and clustered hints)
- Out-of-scope features (record-file format, storage layout, UI widgets, query parameters) — these stay in the backend/tooling
- Target generators and serialization — unchanged scope boundary from the core model

## Key Assumptions to Validate

| Tier | Assumption | How to validate |
|------|------------|-----------------|
| Must-be-true | These five features can be added to the existing model without reshaping the v0 concepts (Entity/Collection/Recordset/property/field/column). | Prototype each in polymodel-go against the existing types; confirm no breaking change to v0. |
| Must-be-true | Localized types and map/dictionary types fit a single coherent type-system extension (not five ad-hoc additions). | Design the type system reconciliation first; confirm localization is a special case of the generic map. |
| Should-be-true | A collection-level primary key composes with field-level bindings/foreign keys without ambiguity. | Model a real inGitDB collection (with `primary_key`) and a DataTug table in the extended types. |
| Should-be-true | Computed/derived fields can be expressed without committing to a formula language now (mirroring the DTQL query seam). | Decide whether `formula` uses an opaque seam like the query, or a constrained expression. |
| Might-be-true | Nested document schema can reuse the existing property/field model recursively rather than a separate JSON-Schema dialect. | Sketch a `document`-typed attribute whose shape is described with PolyModel fields. |


## SpecScore Integration

- **New Features this would create:** likely one Feature per gap (localized-types,
  map-types, collection-key, computed-fields, document-schema), or a single
  `type-system-v1` Feature that bundles the type-system gaps (localization, maps,
  document schema) with collection-key and computed-fields as siblings. Decide at
  specify time.
- **Existing Features affected:** the `core-model` Feature tree (parent and
  sub-features) — additive only; v0 behavior is preserved.
- **Dependencies:** the `core-model` Feature and its Decisions (especially the
  deferred type-system reconciliation noted in `core-model` and `polymodel-go`).
  Source analysis: `spec/research/parity-datatug-ingitdb.md`.

## Open Questions

- Does localization generalize from the map/dictionary type (localized = `map[locale]
  string`), or warrant a dedicated concept?
- Should computed fields use an opaque expression seam (like the DTQL query seam) or a
  constrained, validated expression language?
- Is a collection-level primary key a distinct concept, or does it reuse the Entity
  `key` mechanism applied to a collection's fields?
- Should a nested `document` schema reuse the property/field model recursively, or
  reference a separate schema artifact?
