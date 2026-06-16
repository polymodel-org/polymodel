---
format: https://specscore.md/plan-specification
status: Completed
---
# Plan: Recordset

**Status:** Completed
**Source Feature:** core-model/recordset
**Date:** 2026-06-16
**Owner:** alexandertrakhimenok
**Supersedes:** —

## Summary

Implement the Recordset-specific behavior of `polymodel-go` v0: the tabular result
shape with strict, ordered columns — including a `document`-typed column, a column with
a source expression, and a column with a soft navigational reference — plus a declared
key. Builds on the `core-model` foundations plan.

## Approach

Test-driven Go, sequenced **after** the `core-model` foundations plan (Recordset types,
parser, query seam, and validation come from there). This plan adds the Recordset
result-shape behavior and its tests.

## Tasks

### Task 1: Recordset — strict columns, keys, source expr, soft nav ref

**Verifies:** core-model/recordset#ac:recordset-shape
**Depends-On:** —
**Status:** done

Implement the Recordset as a result shape with ordered, strict columns — a
`document`-typed column, a column with a source expression, and a column with a soft
navigational reference — plus a declared key. Tests assert column order is preserved
and the soft reference is recorded but not enforced as a constraint.

## Open Questions

None at this time.

---
*This document follows the https://specscore.md/plan-specification*
