---
format: https://specscore.md/plan-specification
status: Approved
---
# Plan: Collection

**Status:** Approved
**Source Feature:** core-model/collection
**Date:** 2026-06-16
**Owner:** alexandertrakhimenok
**Supersedes:** —

## Summary

Implement the Collection-specific behavior of `polymodel-go` v0: the `Kind`
(Editable|Computed) data source, loose schemaless-capable fields with bindings and
enforced foreign keys, and the Computed-collection query rule. Builds on the
`core-model` foundations plan.

## Approach

Test-driven Go, sequenced **after** the `core-model` foundations plan (Collection types,
parser, query seam, and validation come from there). Linear order: kind, then fields,
then the computed-query rule.

## Tasks

### Task 1: Collection — Kind and data source

**Verifies:** core-model/collection#ac:collection-kind
**Depends-On:** —
**Status:** pending

Implement the Collection as a data source with a required `Kind` of `Editable` or
`Computed`. Tests assert both kinds validate and that a Collection whose Kind is
neither is rejected with a located error.

### Task 2: Collection fields — loose, optional, binding, enforced FK

**Verifies:** core-model/collection#ac:collection-fields
**Depends-On:** 1
**Status:** pending

Implement loose, schemaless-capable fields: an `Optional` field, a field bound to an
Entity property, and a field with an enforced foreign key to another Collection. Tests
assert binding and FK references resolve and that an FK to a missing Collection is
rejected with a located error.

### Task 3: Computed collections — query required, Editable forbidden

**Verifies:** core-model/collection#ac:computed-query
**Depends-On:** 1
**Status:** pending

Enforce that a Computed Collection carries a query (via the foundations query seam) and
an Editable one does not. Tests assert a Computed Collection missing a query — or an
Editable Collection carrying one — is rejected with a located error.

## Open Questions

None at this time.

---
*This document follows the https://specscore.md/plan-specification*
