---
format: https://specscore.md/feature-specification
status: Approved
---

# Feature: PolyModel Recordset

> [SpecScore.**Studio**](https://specscore.studio): | [Explore](https://specscore.studio/app/github.com/polymodel-org/polymodel/spec/features/core-model/recordset?op=explore) | [Edit](https://specscore.studio/app/github.com/polymodel-org/polymodel/spec/features/core-model/recordset?op=edit) | [Ask question](https://specscore.studio/app/github.com/polymodel-org/polymodel/spec/features/core-model/recordset?op=ask) | [Request change](https://specscore.studio/app/github.com/polymodel-org/polymodel/spec/features/core-model/recordset?op=request-change) |
**Status:** Approved
**Source Ideas:** —

## Summary

Recordset: the tabular shape of a query or stored-procedure result, with strict columns.

## Problem

A Recordset is the shape of a **result** — what a query or stored procedure returns —
as distinct from a stored Collection. Modeling it separately lets PolyModel describe a
stored-procedure's output, or a query result, without inventing a container. Unlike a
Collection's loose fields, a result is strictly tabular: every row has the same
columns. This sub-feature specifies the Recordset and its strict `column` attribute;
shared rules (attribute types, binding precedence, type system, query seam, validation)
live in the parent [core-model](../README.md).

## Behavior

### Result shape

#### REQ: recordset-shape

A **Recordset** is the **tabular shape of a result** — the output of a query or a
stored procedure / function. It is NOT a `FROM`-able named source. A Recordset MUST
consist of ordered **columns**, MAY declare **keys** (a logical primary/alternate key
of the result), and MAY carry a **query** behind the parent's `query-seam`.

#### REQ: recordset-column

A Recordset **column** is strict and tabular: every row of the Recordset has the same
columns. A column MUST have a name and a type; its type MAY be a `document`/JSON value.
A column MAY **bind** to an Entity property (semantic meta, subject to the parent's
`binding-local-primary` rule), MAY carry a **source expression** (what produced it), and
MAY declare a **soft navigational reference** for lookups (unlike a Collection foreign
key, it is not enforced).

## Acceptance Criteria

### AC: recordset-shape
**Requirements:** recordset#req:recordset-shape, recordset#req:recordset-column

**Scenario:** A result shape with ordered, strict columns.
**Given** a Recordset with ordered columns — one typed `document`, one carrying a source expression, one declaring a soft navigational reference — and a declared key
**When** the model is validated
**Then** it validates as a result shape, the column order is preserved, and the soft navigational reference is recorded but not enforced as a constraint.

## Open Questions

None at this time.

---
*This document follows the https://specscore.md/feature-specification*
