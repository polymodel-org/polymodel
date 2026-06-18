---
format: https://specscore.md/decision-specification
status: Approved
---

# Decision: Attribute model: property, field, column

**Status:** Approved
**Date:** 2026-06-16
**Owner:** alex
**Tags:** —
**Source Idea:** core-model
**Supersedes:** —
**Superseded By:** —

## Context

While specifying the core model we had to decide how attributes are modeled across
the three concepts — Entity, Collection, Recordset. The question was whether they
share one attribute type or are distinct. What was known: Collections can be
schemaless / document-oriented (Git, Firestore), Recordsets are strictly tabular
results, and Entities are the canonical semantic layer the others bind back to.

## Decision

Use three distinct attribute types, named for their layer: an Entity has
**properties**, a Collection has **fields**, a Recordset has **columns**.

## Rationale

The real cleavage is semantic vs loose-container vs strict-tabular. "Column" carries
the RDBMS connotation of a fixed schema every record conforms to — wrong for a
schemaless Collection, where "field" is the fluent term. A Recordset is tabular by
definition, so "column" is precise there, and using a different word from "field"
keeps "loose container" and "tabular result" visibly distinct. The Entity attribute is
the only canonical, authoritative one (it owns identity and true constraints; the
others bind to it), which "property" names. Distinct names make the model self-
documenting and prevent a single type from accreting contradictory concerns.

## Declined Alternatives

### One merged tabular attribute type

Treat Collection and Recordset attributes as a single columnar type. Lost: a Collection
can be schemaless or hold heterogeneous documents, so a fixed-column model is wrong for
it; a loose field and a strict column are genuinely different.

### A single shared term (`field`) for all three

Use `field` uniformly across Entity, Collection, and Recordset. Lost: it erases the
meaningful distinction between the canonical semantic attribute, a loose container
attribute, and a strict tabular result attribute — the names are doing real work.

## Consequences at Decision Time

Three attribute types to implement and keep coherent (slightly more surface than one).
A clearer mental model and self-documenting spec. Binding is modeled as field/column →
property references. Generators and tooling can treat the three layers differently
where that matters.

## Observed Consequences

None observed yet.

## Affected Features

- core-model
- core-model/entity
- core-model/collection
- core-model/recordset

---
*This document follows the https://specscore.md/decision-specification*
