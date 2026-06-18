---
format: https://specscore.md/decision-specification
status: Approved
---

# Decision: DTQL behind a placeholder query seam

**Status:** Approved
**Date:** 2026-06-16
**Owner:** alex
**Tags:** —
**Source Idea:** core-model
**Supersedes:** —
**Superseded By:** —

## Context

Computed Collections (views) and Recordsets need a way to express the query that
produces their rows. DTQL — DataTug's dialect-agnostic query AST — is the intended
language. What was known at decision time: `dtql-go` is an empty repository (a README
only, no AST), so there is no concrete DTQL type to depend on yet.

## Decision

Represent the query body behind a **placeholder seam** — an opaque query reference the
model carries without interpreting. DTQL is the intended AST and will be swapped in
behind the seam once `dtql-go` has a stable one; v0 MUST NOT depend on a concrete DTQL
implementation.

## Rationale

The seam unblocks v0 model and validation work without taking a hard dependency on an
unfinished repository, while still committing to DTQL as the direction. Because the
model never interprets the query in v0, a real DTQL AST can replace the placeholder
later without reshaping Collection or Recordset.

## Declined Alternatives

### Depend on `dtql-go` now

Import the DTQL AST directly. Lost: `dtql-go` is empty, so v0 would block on building
that AST in another repository before any model work could land.

### Define a PolyModel-native query AST

Build PolyModel's own query language. Lost: it duplicates DTQL, which already targets
exactly this problem.

### Defer query bodies entirely

Model only shapes, with no query representation. Lost: a Computed Collection / view
could not express what computes it, leaving a hole in the model.

## Consequences at Decision Time

Computed Collections and Recordsets carry an opaque query value in v0; a later change
swaps a DTQL AST behind the seam. There is a risk the seam's shape needs revision when
DTQL lands, mitigated by keeping it opaque (no structure to break). No `dtql-go`
dependency in v0.

## Observed Consequences

None observed yet.

## Affected Features

- core-model
- core-model/collection
- core-model/recordset

---
*This document follows the https://specscore.md/decision-specification*
