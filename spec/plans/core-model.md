---
format: https://specscore.md/plan-specification
status: Implemented
---
# Plan: Core Model

**Status:** Implemented
**Source Feature:** core-model
**Date:** 2026-06-16
**Owner:** alexandertrakhimenok
**Supersedes:** —

## Summary

Implement the `polymodel-go` v0 **foundations** — the in-memory model types, the HCL
parser, the opaque query seam, binding semantics, Component scoping, type/constraint
validation, and model-wide reference resolution. Code lands in the separate
`polymodel-org/polymodel-go` module (currently a scaffold). This plan covers the parent
`core-model` Feature's seven acceptance criteria. The per-concept behavior is covered by
sibling plans: `entity`, `collection`, and `recordset`.

## Approach

Test-driven Go. This foundations plan is sequenced **first**; the three concept plans
depend on it (they build on the model types and the parser). Within this plan: define
the in-memory types, then the HCL parser (so later behavior can be exercised from HCL
fixtures), then the query seam, binding, Component scoping, and finally type/constraint
and reference/located validation. Order is strictly linear; each task depends on its
predecessors.

Decisions applied: D-0001 (three attribute types — property/field/column), D-0002
(`Collection.Kind` Editable|Computed, no `Table` type), D-0003 (opaque query seam, no
`dtql-go` dependency), D-0004 (HCL-first via `hashicorp/hcl/v2`; consumers use the
in-memory model directly).

Out of scope: target generators, JSON/YAML serialization, catalog / `@pm:` references,
and migrations.

## Tasks

### Task 1: In-memory model types

**Verifies:** core-model#ac:model-concepts
**Depends-On:** —
**Status:** done

Define the Go types for the three structural concepts and their distinct attribute
types — `Entity`/`Property`, `Collection`/`Field` (with `Kind`), `Recordset`/`Column` —
plus `Component`, `TypeRef`, and `Constraints`. Tests assert each concept exposes its
own attribute type and that names are unique within a concept kind.

### Task 2: HCL parsing into the model

**Verifies:** core-model#ac:authoring, core-model#ac:model-concepts
**Depends-On:** 1
**Status:** done

Parse HCL (via `hashicorp/hcl/v2`) into the in-memory model so a Go consumer uses it
directly with no serialized interchange file. Tests parse an HCL model declaring an
Entity, Collection, and Recordset, and assert an unknown top-level concept is rejected
with a located error.

### Task 3: Opaque query seam

**Verifies:** core-model#ac:query-seam-opaque
**Depends-On:** 2
**Status:** done

Implement the placeholder query seam — an opaque query reference carried by Computed
Collections and Recordsets without interpretation. Tests assert a model carrying a query
loads and retains it verbatim, with no concrete DTQL implementation present.

### Task 4: Binding precedence — local attribute primary

**Verifies:** core-model#ac:binding-precedence
**Depends-On:** 2
**Status:** done

Implement binding semantics so a Collection field / Recordset column that binds to an
Entity property is primary and may override the property (e.g. nullability). Tests
assert the override applies, the binding resolves, and an unbound field/column is valid.

### Task 5: Component scope — Entities only

**Verifies:** core-model#ac:component-scope
**Depends-On:** 2
**Status:** done

Enforce that Components embed into Entities only in v0. Tests assert an Entity embed
validates and a Collection (or Recordset) embed is rejected with a located error.

### Task 6: Type system and constraint validation

**Verifies:** core-model#ac:type-system
**Depends-On:** 1
**Status:** done

Validate that every property/field/column type is in the v0 primitive set (or a valid
reference) and that constraints are within the v0 set (`required`, `unique`, length,
`pattern`, `enum`). Tests assert in-set types/constraints validate and out-of-set ones
are rejected with located errors.

### Task 7: Reference resolution and located validation

**Verifies:** core-model#ac:validation-located
**Depends-On:** 3, 4, 5, 6
**Status:** done

Implement model-wide reference resolution (property reuse, Component embeds, Entity
associations, Collection foreign keys, field/column→property bindings) and a validation
pass that emits located errors (object + attribute context). Tests assert an unresolved
reference fails with a located error naming the offending object and attribute.

## Open Questions

None at this time.

---
*This document follows the https://specscore.md/plan-specification*
