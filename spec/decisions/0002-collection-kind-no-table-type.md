---
format: https://specscore.md/decision-specification
status: Approved
---

# Decision: No Table type — Collection.Kind (Editable or Computed)

**Status:** Approved
**Date:** 2026-06-16
**Owner:** alex
**Tags:** —
**Source Idea:** core-model
**Supersedes:** —
**Superseded By:** —

## Context

The core model needs to represent both stored containers (tables / file-system
collections) and views. The question was whether tables and views are distinct
structural types. What was known: to a consumer, a view is indistinguishable from a
table at `SELECT FROM` time — both are named data sources you read from.

## Decision

There is no separate `Table` type. A single **Collection** carries a **Kind** attribute
of either `Editable` (a stored table / FS collection) or `Computed` (a view).

## Rationale

Modeling tables and views as one Collection with a Kind keeps the consumer contract
uniform: tooling reads `FROM` a Collection without caring whether it is stored or
computed. The Kind is partly derivable (a Collection with a query is Computed), but an
explicit Kind is clearer and cheap, and it lets validation enforce kind-specific rules
(Computed must carry a query; Editable must not). Two separate types would duplicate
the data-source concept for no consumer benefit.

## Declined Alternatives

### Separate `Table` and `View` types

Model tables and views as distinct structural types. Lost: it duplicates the
data-source concept, and a view behaves exactly like a table at query time, so the
distinction belongs on an attribute, not in the type system.

## Consequences at Decision Time

A Collection carries a `Kind`; Computed collections additionally carry a query (behind
the query seam); both kinds expose the same uniform `field` model. Validation enforces
the Editable-vs-Computed query rule. Future kinds (if ever needed) extend the enum
rather than adding types.

## Observed Consequences

None observed yet.

## Affected Features

- core-model
- core-model/collection

---
*This document follows the https://specscore.md/decision-specification*
