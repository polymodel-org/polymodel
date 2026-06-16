---
format: https://specscore.md/feature-specification
status: Approved
---

# Feature: PolyModel Collection

> [SpecScore.**Studio**](https://specscore.studio): | [Explore](https://specscore.studio/app/github.com/polymodel-org/polymodel/spec/features/core-model/collection?op=explore) | [Edit](https://specscore.studio/app/github.com/polymodel-org/polymodel/spec/features/core-model/collection?op=edit) | [Ask question](https://specscore.studio/app/github.com/polymodel-org/polymodel/spec/features/core-model/collection?op=ask) | [Request change](https://specscore.studio/app/github.com/polymodel-org/polymodel/spec/features/core-model/collection?op=request-change) |
**Status:** Approved
**Source Ideas:** —

## Summary

Collection: a named, FROM-able data source (Editable table/FS-collection or Computed view) with loose, schemaless-capable fields.

## Problem

A Collection is where records actually live or are computed — the thing a consumer
reads `FROM`. PolyModel must model both stored containers and views *without* forcing a
rigid columnar schema (document and Git/Firestore collections are schemaless-capable)
and without splitting tables and views into separate types (a view is indistinguishable
from a table at query time). This sub-feature specifies the Collection and its loose
`field` attribute; shared rules (attribute types, binding precedence, type system,
query seam, validation) live in the parent [core-model](../README.md).

## Behavior

### Data source and kind

#### REQ: collection-data-source

A **Collection** is a named, addressable, queryable data source that holds records —
the thing a consumer reads `FROM`. A Collection's attributes are **fields**.

#### REQ: collection-kind

A Collection MUST declare a **Kind** of either `Editable` (a stored table / file-system
collection) or `Computed` (a view). There is NO separate `Table` or `View` concept —
the distinction is the `Kind` attribute, so a consumer treats both uniformly as data
sources. A Collection whose Kind is neither value is invalid.

### Fields

#### REQ: collection-field

A Collection **field** is loose and schemaless-capable: a record in the Collection MAY
carry fewer or more fields than declared, and a field MAY be marked **Optional**. A
field MAY **bind** to an Entity property by reference (inheriting its semantics, subject
to the parent's `binding-local-primary` rule), MAY exist with no Entity behind it, and
MAY declare an **enforced** foreign key to another Collection.

### Computed collections

#### REQ: collection-computed-query

A Collection of Kind `Computed` MUST carry a **query** that defines its content; a
Collection of Kind `Editable` MUST NOT. The query is held behind the parent's
`query-seam` (an opaque query reference) in v0.

## Acceptance Criteria

### AC: collection-kind
**Requirements:** collection#req:collection-data-source, collection#req:collection-kind

**Scenario:** Kind is required and constrained.
**Given** one Collection with Kind `Editable` and another with Kind `Computed`
**When** the model is validated
**Then** both validate as data sources, and a Collection whose Kind is neither `Editable` nor `Computed` is rejected with a located error.

### AC: collection-fields
**Requirements:** collection#req:collection-field

**Scenario:** Loose fields, binding, and enforced foreign keys.
**Given** a Collection with an Optional field, a field bound to an Entity property, and a field declaring an enforced foreign key to another Collection
**When** the model is validated
**Then** the fields validate and the property binding and foreign-key references resolve, and a foreign key targeting a Collection that does not exist is rejected with a located error.

### AC: computed-query
**Requirements:** collection#req:collection-computed-query

**Scenario:** Computed carries a query; Editable does not.
**Given** a Computed Collection that carries a query and an Editable Collection that carries none
**When** the model is validated
**Then** both validate, and a Computed Collection missing a query — or an Editable Collection carrying one — is rejected with a located error.

## Open Questions

None at this time.

---
*This document follows the https://specscore.md/feature-specification*
